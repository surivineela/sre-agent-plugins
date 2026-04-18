---
name: service-ownership-lookup
description: "Look up service ownership, oncall contacts, SLA tiers, and escalation paths from Contoso's internal CMDB. Use during any incident to determine who owns the affected service, what SLA applies, and who to escalate to."
---

# Service Ownership Lookup — Contoso CMDB

You are looking up service ownership information from Contoso's internal CMDB. Use the `contoso-cmdb` MCP connector for all queries. This information is NOT available from Kubernetes labels, Azure tags, or any Azure-native API.

## CMDB API

Use the `contoso-cmdb` MCP connector.

- `GET /services?namespace={namespace}` — Look up by K8s namespace
- `GET /services?resource_id={azure_resource_id}` — Look up by Azure resource
- `GET /services?name={service_name}` — Fuzzy match by name
- `GET /teams/{team}/services` — All services for a team

## SLA Tier Reference

| Tier | Response | Resolution | Examples | Escalation |
|------|----------|-----------|----------|------------|
| Tier-0 | 15 min | 4 hours | Payments, Identity/Auth | VP at 60 min, CTO at 120 min |
| Tier-1 | 30 min | 8 hours | Marketplace, Data Platform | Manager at 2 hours |
| Tier-2 | 4 hours | 24 hours | Batch processing, internal tools | Team lead next business day |
| Tier-3 | Next business day | 5 business days | Documentation, non-critical dashboards | N/A |

## Safety Rules

- NEVER share oncall personal phone numbers in investigation reports — use PagerDuty IDs
- ALWAYS check SLA tier before deciding response urgency
- If SLA is breached, note it prominently in your report
- Reference previous incidents when patterns match