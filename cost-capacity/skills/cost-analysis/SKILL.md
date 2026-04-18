---
name: cost-analysis
description: "Contoso-specific cost analysis using the internal cost dashboard. Provides team-level spend, budget burn rates, chargeback attribution, and variance explanations. Use when investigating cost spikes, reviewing team budgets, or assessing the cost impact of infrastructure decisions."
---

# Cost Analysis — Contoso FinOps

You are performing cost analysis using Contoso's internal cost dashboard and FinOps policies. Use the `contoso-cost-dashboard` MCP connector for all cost data queries — it has team-level data that Azure Cost Management does not.

## Contoso Cost Model

### Chargeback Structure
Costs at Contoso are attributed by **Kubernetes namespace label** and **Azure resource tag**.

### Budget Tiers

| Team Category | Monthly Budget | Alert Threshold | Escalation |
|---------------|---------------|-----------------|------------|
| Tier-0 services (Payments, Identity) | $50,000 | 80% at mid-month | Budget owner + VP-Eng |
| Tier-1 services (Marketplace, Data) | $30,000 | 80% at mid-month | Budget owner |
| Tier-2 services (Batch, Internal tools) | $10,000 | 90% at month-end | Budget owner |
| Development/test | $5,000 per team | 100% hard cap | Auto-scale disabled at cap |

## Cost Dashboard API

Use the `contoso-cost-dashboard` MCP connector.

- `GET /teams/{team}/spend?period={period}` — Team spend with budget status
- `GET /teams/{team}/variance?compare=previous-month` — Cost variance
- `GET /resources?resource_group={rg}&sort=cost_desc&limit=20` — Resource costs

## Safety Rules

- NEVER recommend scaling down Tier-0 services purely for cost savings during business hours
- ALWAYS show the budget impact before recommending any resource change
- Flag any unattributed resources (missing cost tags) — these are policy violations