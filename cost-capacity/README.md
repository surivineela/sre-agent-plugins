# Cost & Capacity Plugin

**Author**: Cloud FinOps Team (finops@contoso.com)
**Version**: 1.0.0

## Overview

This plugin provides Contoso-specific FinOps policies, SKU governance rules, and integration with our internal cost dashboard. It turns generic "you could save money" suggestions into actionable, org-specific recommendations that respect Contoso's approval workflows and cost allocation model.

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| `cost-analysis` | Team-level cost analysis with budget burn rates, chargeback attribution, and variance explanations |
| `capacity-planning` | SKU approval matrix, scaling governance (scale-out vs scale-up rules), and capacity threshold policies |

### MCP Connectors

| Connector | Description |
|-----------|-------------|
| `contoso-cost-dashboard` | Queries Contoso's internal cost dashboard for team-level spend, budget data, and chargeback reports |

## Why This Exists

The agent can run `az cost management query` natively. This plugin adds what Azure Cost Management doesn't know:

1. **Who pays for what**: Our chargeback model assigns costs to teams by namespace label, not just subscription.
2. **Approval authority**: D-series above D4 requires VP approval. Scale horizontally instead to stay within team-level authority.
3. **Budget context**: Our dashboard shows burn rate against budget, forecasted overage, and who the budget owner is.

## Required Setup

1. Obtain a read-only API token for the cost dashboard from the FinOps team
2. When installing this plugin, enter the token when prompted for the `contoso-cost-dashboard` connector