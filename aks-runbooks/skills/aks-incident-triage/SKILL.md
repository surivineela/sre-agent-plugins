---
name: aks-incident-triage
description: "Contoso-specific AKS incident triage runbook. Follows org-specific troubleshooting procedures: node pool naming conventions, PDB-first restart policy, namespace-based escalation paths, and known failure pattern matching. Use when investigating pod failures, node pressure, or AKS cluster issues."
---

# AKS Incident Triage — Contoso Runbook

You are triaging an AKS incident using Contoso's Kubernetes Platform Team runbooks. Follow Contoso's procedures exactly — do NOT use generic Kubernetes troubleshooting steps.

## Contoso AKS Conventions

### Cluster Naming
- Production: `aks-{team}-prod-{region}` (e.g., `aks-payments-prod-eastus2`)
- Staging: `aks-{team}-staging-{region}`
- Dev: `aks-{team}-dev-{region}`

### Node Pool Naming & Purpose
| Pool Name Pattern | Purpose | Expected SKU | Auto-scale |
|-------------------|---------|-------------|------------|
| `system-*` | System pods (CoreDNS, kube-proxy, metrics-server) | Standard_D2s_v5 | 1–3 nodes |
| `user-*` | Application workloads | Standard_B4ms or Standard_D4s_v5 | 2–10 nodes |
| `gpu-*` | ML inference workloads | Standard_NC6s_v3 | 0–4 nodes |
| `spot-*` | Batch processing, non-critical jobs | Standard_D4s_v5 (Spot) | 0–20 nodes |

### Namespace Conventions
| Namespace Pattern | Team | Escalation Contact | SLA |
|-------------------|------|-------------------|-----|
| `payments-*` | Payments Platform | @payments-oncall | Tier-0 (15 min) |
| `marketplace-*` | Marketplace Backend | @marketplace-oncall | Tier-1 (30 min) |
| `identity-*` | Identity & Auth | @identity-oncall | Tier-0 (15 min) |
| `data-*` | Data Platform | @data-oncall | Tier-1 (30 min) |
| `batch-*` | Batch Processing | @batch-team | Tier-2 (4 hours) |
| `platform-*` | Platform Engineering | @k8s-platform | Tier-1 (30 min) |

## Triage Procedure

### Step 1: Identify Scope

```bash
# Check cluster health
kubectl get nodes -o wide
kubectl top nodes

# Identify affected pods
kubectl get pods --all-namespaces --field-selector status.phase!=Running

# Check recent events (last 30 minutes)
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -50
```

Determine the affected namespace and use the table above to identify the owning team and SLA tier.

### Step 2: Classify the Incident

#### OOMKilled Pods
**Contoso-specific**: Before recommending memory limit increases, check:
1. Was there a recent deployment? (Query `contoso-deploy-tracker`: `GET /deployments?namespace={ns}&since=2h`)
2. If yes, compare the previous deployment's resource requests/limits to the current one
3. Check if the memory increase pattern matches known issue **K8S-2024-017** (Java apps with default heap after JDK 17 upgrade — set `-XX:MaxRAMPercentage=75`)

```bash
# Check current resource settings
kubectl describe pod {pod} -n {namespace} | grep -A 5 "Limits\|Requests"

# Check previous deployment's settings via deploy tracker
# Use MCP connector: GET /deployments/{namespace}/{deployment}/diff?compare=previous
```

**IMPORTANT — Contoso PDB Policy**: Before restarting ANY pod in a `payments-*` or `identity-*` namespace:
1. Verify a PodDisruptionBudget exists: `kubectl get pdb -n {namespace}`
2. If no PDB exists, DO NOT restart. Escalate to the owning team first.
3. If PDB exists, verify `minAvailable` >= 2 before proceeding with restart.

#### Node Pressure (Memory/Disk/PID)
**Contoso-specific procedure**:
1. Check if the node is in a `system-*` pool — if yes, this is **Tier-0** regardless of namespace
2. Check if node has taint `contoso.com/drain-scheduled=true` — if yes, a planned drain is in progress, this is expected
3. Verify the Resource Quota for the affected namespace: `kubectl describe resourcequota -n {namespace}`
4. If node memory utilization > 85%, check for pods without memory limits (violates Contoso policy CKP-003)

```bash
# Find pods without memory limits (policy violation)
kubectl get pods -n {namespace} -o json | jq '.items[] | select(.spec.containers[].resources.limits.memory == null) | .metadata.name'
```

#### ImagePullBackOff
**Contoso-specific**: All images must come from our ACR:
- Production: `contosoprod.azurecr.io`
- Non-production: `contosodev.azurecr.io`

Check:
1. Is the image tag using `latest`? (Violates Contoso policy CKP-001 — all prod images must use SHA digests or semver tags)
2. Is the image from the correct ACR for the environment?
3. Is the ACR pull secret `acr-pull-secret` present in the namespace?

```bash
kubectl get secret acr-pull-secret -n {namespace}
kubectl describe pod {pod} -n {namespace} | grep "Image:"
```

#### CrashLoopBackOff
1. Pull container logs: `kubectl logs {pod} -n {namespace} --previous`
2. Query deployment tracker for recent changes: `GET /deployments?namespace={ns}&since=4h`
3. If the crash pattern matches known issue **K8S-2024-022** (health probe timeout too aggressive after adding a sidecar), recommend increasing `initialDelaySeconds` to 30

### Step 3: Correlate with Deployments

**Always** query the deployment tracker to check for recent changes:

```
# Via MCP connector (contoso-deploy-tracker)
GET /deployments?namespace={namespace}&since=2h

# Response includes:
# - deployment_id, timestamp, deployer, team
# - image_tag (previous and current)
# - resource_diff (what changed in the manifest)
# - rollback_command (pre-generated rollback)
```

If a deployment occurred within the incident window:
1. Compare the resource diff to the current symptoms
2. If there's a correlation, recommend rollback using the pre-generated command
3. Tag the incident as `deployment-correlated` for postmortem tracking

### Step 4: Remediation Decision Tree

```
Is this in a Tier-0 namespace (payments-*, identity-*)?
├── YES → Does a PDB exist?
│   ├── YES → Is the PDB satisfied (enough healthy replicas)?
│   │   ├── YES → Safe to restart affected pods
│   │   └── NO → Scale up replicas first, THEN restart
│   └── NO → DO NOT restart. Page the owning team's oncall.
└── NO → Is this in a spot-* pool?
    ├── YES → Spot eviction. Expected behavior. Check if the workload has proper node affinity and tolerations.
    └── NO → Standard restart procedure:
        1. kubectl rollout restart deployment/{name} -n {namespace}
        2. Monitor: kubectl rollout status deployment/{name} -n {namespace}
        3. Verify: kubectl get pods -n {namespace} -l app={name}
```

## Known Issues Registry

| Issue ID | Pattern | Root Cause | Fix |
|----------|---------|-----------|-----|
| K8S-2024-017 | OOMKill on Java pods after JDK upgrade | Default heap sizing changed in JDK 17 | Add `-XX:MaxRAMPercentage=75` to JAVA_OPTS |
| K8S-2024-022 | CrashLoop after sidecar addition | Liveness probe fires before sidecar is ready | Increase `initialDelaySeconds` to 30 |
| K8S-2024-031 | Node NotReady after scale-out | kubelet takes >2min on D-series with encrypted disks | Increase node readiness timeout to 180s |
| K8S-2025-004 | Intermittent DNS failures | CoreDNS on system pool hits memory limit | Increase CoreDNS memory limit to 256Mi |

## Safety Rules

- NEVER delete a PDB in a production namespace
- NEVER drain a system-pool node without verifying CoreDNS has replicas on other nodes
- NEVER scale down below the PDB's `minAvailable` threshold
- ALWAYS query the deployment tracker before recommending code-level changes
- For Tier-0 services, page the owning team even if you can fix it autonomously