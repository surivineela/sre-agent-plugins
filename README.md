# Contoso SRE Plugins — Internal Plugin Marketplace

This GitHub repository serves as **Contoso's internal plugin marketplace** for Azure SRE Agent. It's maintained by the SRE Platform Team and contains curated operational knowledge and internal API integrations contributed by specialist teams across the organization.

## The Problem This Solves

The SRE Agent is smart — it can run `kubectl`, query Azure Monitor, and troubleshoot generic Kubernetes issues. But it doesn't know **your organization's** tribal knowledge:

- Your node pool naming conventions and PDB policies
- Your SKU approval matrix and cost allocation model
- Your service ownership, SLA tiers, and escalation paths
- Your internal APIs (deployment tracker, cost dashboard, CMDB)

These plugins bridge the gap between **generic intelligence** and **org-specific operational expertise**.

## Available Plugins

| Plugin | Author Team | Skills | MCP Connector | Category |
|--------|------------|--------|---------------|----------|
| [AKS Runbooks](./aks-runbooks/) | Kubernetes Platform | `aks-incident-triage`, `aks-deployment-analysis` | Internal Deployment Tracker | Operations |
| [Cost & Capacity](./cost-capacity/) | Cloud FinOps | `cost-analysis`, `capacity-planning` | Internal Cost Dashboard | Cost Management |
| [Service Catalog](./service-catalog/) | SRE Leadership | `service-ownership-lookup`, `dependency-impact-analysis` | Internal CMDB | Governance |

**Total**: 3 plugins, 6 skills, 3 MCP connectors to internal APIs

## What Each Plugin Brings

| Without Plugin | With Plugin |
|---------------|-------------|
| "Pod is OOMKilled — increase memory limit" | "The 14:32 deploy by team-payments changed memory request without updating limit. Matches known issue K8S-2024-017." |
| "Consider a larger VM SKU" | "D8s_v3 needs VP approval. Scale horizontally instead — +$115/month, within team authority. Budget: $18,800 remaining." |
| Agent doesn't know who to notify | "Tier-0 service, 15-min SLA. Paging @payments-oncall. 3 downstream services affected including fraud scoring (allow-all fallback — SECURITY RISK)." |

## How SRE Teams Use This

1. **Register** this marketplace in your SRE Agent instance (one-time setup)
2. **Browse** the catalog to see what plugins are available from different teams
3. **Install** the plugins your team needs — each install brings skills + an internal API connector
4. **Use** — the agent's responses become org-specific and actionable
5. **Refresh** — when a team updates their plugin, pull the latest with one click

## Repository Structure

```
.github/
  plugin/
    marketplace.json             ← SRE Agent discovers this file
aks-runbooks/                    ← Plugin 1 (by K8s Platform Team)
  .mcp.json                        Connector: Internal Deployment Tracker
  README.md
  skills/
    aks-incident-triage/SKILL.md
    aks-deployment-analysis/SKILL.md
cost-capacity/                   ← Plugin 2 (by FinOps Team)
  .mcp.json                        Connector: Internal Cost Dashboard
  README.md
  skills/
    cost-analysis/SKILL.md
    capacity-planning/SKILL.md
service-catalog/                 ← Plugin 3 (by SRE Leadership)
  .mcp.json                        Connector: Internal CMDB
  README.md
  skills/
    service-ownership-lookup/SKILL.md
    dependency-impact-analysis/SKILL.md
```