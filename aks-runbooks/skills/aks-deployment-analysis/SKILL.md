---
name: aks-deployment-analysis
description: "Correlate AKS incidents with recent deployments by querying Contoso's internal deployment tracker. Identifies what changed, who deployed it, and whether a rollback is recommended. Use when an AKS incident may be caused by a recent deployment."
---

# AKS Deployment Analysis — Contoso

You are analyzing whether a recent deployment caused or contributed to an AKS incident. Use Contoso's internal deployment tracker (via the `contoso-deploy-tracker` MCP connector) to get deployment history that is NOT available via kubectl.

## Why This Skill Exists

`kubectl rollout history` only shows the Kubernetes deployment revision. It doesn't tell you:
- **Who** triggered the deployment (person or CI pipeline)
- **What** changed in the application code (not just the manifest)
- **Why** it was deployed (linked Jira ticket, PR, or incident)
- **Whether** it passed Contoso's deployment gates (canary, integration tests, approval)

The deployment tracker has all of this.

## Deployment Tracker API

Use the `contoso-deploy-tracker` MCP connector for all queries.

### Get Recent Deployments

```
GET /deployments?namespace={namespace}&since={duration}

# Example: GET /deployments?namespace=payments-prod&since=4h
```

### Get Deployment Diff

```
GET /deployments/{namespace}/{deployment_name}/diff?compare=previous
```

### Get Deployment Health

```
GET /deployments/{deployment_id}/health
```

## Analysis Workflow

### Step 1: Timeline Correlation
Fetch deployments in the incident window (typically `since=4h`). Identify any deployment that occurred within 30 minutes BEFORE the incident start.

### Step 2: Change Impact Assessment
For each correlated deployment, evaluate the `resource_diff`.

### Step 3: Gate Verification
Check `deployment_gates` — flag skipped canary, auto-approved resource changes > 2x, or failed integration tests.

### Step 4: Rollback Decision
Use the pre-generated rollback command from the deployment tracker. Tier-0 services get immediate rollback recommendation.

## Contoso Deployment Policies Referenced

- **CDP-001**: All production deployments must be traceable to a pipeline run
- **CDP-003**: Tier-0 services require canary deployment (minimum 10% traffic, 15-minute bake time)
- **CDP-007**: Resource changes exceeding 2x require manual approval (not auto-approvable)
- **CDP-012**: Rollbacks do not require approval — any SRE can roll back a Tier-0 service during an incident