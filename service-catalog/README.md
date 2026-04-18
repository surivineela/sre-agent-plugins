# Service Catalog Plugin

**Author**: SRE Leadership (sre-leadership@contoso.com)
**Version**: 1.0.0

## Overview

This plugin integrates Contoso's internal Configuration Management Database (CMDB) with the SRE Agent. It provides the organizational context that doesn't exist in Azure resource metadata — who owns a service, what depends on it, what SLA it's under, and who to escalate to.

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| `service-ownership-lookup` | Maps Kubernetes workloads and Azure resources to owning teams, oncall contacts, SLA tiers, and escalation paths |
| `dependency-impact-analysis` | Identifies upstream and downstream dependencies of a service to assess blast radius during incidents |

### MCP Connectors

| Connector | Description |
|-----------|-------------|
| `contoso-cmdb` | Queries Contoso's internal CMDB for service ownership, dependency graphs, and SLA metadata |

## Why This Exists

When a pod is crashing in the `payments-prod` namespace, the agent knows which pod and what error. What it does NOT know:

- **This is a Tier-0 service** with a 15-minute SLA
- **The owner is @payments-oncall** — they need to be paged immediately
- **3 downstream services depend on this** — marketplace checkout, refund processor, and fraud scoring will all be affected
- **The VP to escalate to is Priya Sharma** — if this isn't mitigated in 10 minutes, she needs to know

This information lives in Contoso's CMDB, not in Kubernetes labels or Azure tags.

## Required Setup

1. Obtain a read-only API token for the CMDB from the SRE Leadership team
2. When installing this plugin, enter the token when prompted for the `contoso-cmdb` connector