# AKS Runbooks Plugin

**Author**: Kubernetes Platform Team (k8s-platform@contoso.com)
**Version**: 1.0.0

## Overview

This plugin encodes Contoso's Kubernetes Platform Team's operational knowledge for AKS incident response. It replaces the tribal knowledge that currently lives in wiki pages, Slack threads, and senior engineers' heads with structured, versioned runbooks the SRE Agent can follow.

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| `aks-incident-triage` | Contoso-specific AKS troubleshooting: naming conventions, node pool strategy, PDB policies, known failure patterns |
| `aks-deployment-analysis` | Correlates AKS incidents with recent deployments using the internal deployment tracker |

### MCP Connectors

| Connector | Description |
|-----------|-------------|
| `contoso-deploy-tracker` | Queries Contoso's internal deployment tracking API to see what was deployed, when, and by whom |

## Why This Exists

The SRE Agent can already run `kubectl` commands and read AKS metrics natively. This plugin adds what it **can't** do natively:

1. **Org-specific knowledge**: Our node pool naming conventions (`system-*` vs `user-*` vs `gpu-*`), our PDB requirements, our upgrade cadence, our known failure patterns
2. **Deployment correlation**: The agent can see a pod is crashlooping, but it can't see that team-payments deployed v2.3.1 fifteen minutes ago — that data lives in our deployment tracker, not in Kubernetes
3. **Triage priority**: Our triage procedures differ from generic K8s troubleshooting — we check PDBs before recommending restarts, we verify node taints before scaling, we have specific escalation paths per namespace

## Required Setup

1. Obtain an API token for the deployment tracker from the Platform Engineering team
2. When installing this plugin, enter the token when prompted for the `contoso-deploy-tracker` connector