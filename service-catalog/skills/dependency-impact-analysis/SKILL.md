---
name: dependency-impact-analysis
description: "Analyze service dependency graphs from Contoso's CMDB to assess blast radius during incidents. Identifies upstream and downstream dependencies, calculates impact scope, and recommends communication targets. Use during incidents to understand what else might be affected."
---

# Dependency Impact Analysis — Contoso CMDB

You are analyzing the blast radius of an incident by querying Contoso's service dependency graph. Use the `contoso-cmdb` MCP connector to get dependency data that is NOT available from Kubernetes or Azure.

## CMDB Dependency API

Use the `contoso-cmdb` MCP connector.

- `GET /services/{service_id}/dependencies` — Full upstream/downstream graph
- `GET /services/{service_id}/blast-radius` — Summary roll-up
- `GET /services/{service_id}/dependencies/graph?depth=2` — Visual graph

## Dependency Types

| Type | Meaning | Incident Impact |
|------|---------|----------------|
| **Hard** | Service cannot function without this dependency | Downstream service WILL fail |
| **Soft** | Service has a fallback when dependency is unavailable | Degraded experience, check fallback behavior |
| **Async** | Event-driven, eventual consistency | Data gaps or delayed processing, NOT user-facing |

## Safety Rules

- ALWAYS check for soft dependencies with risky fallback modes (e.g., allow-all for security services)
- Escalate to Security Engineering immediately if any security-related service falls back to a permissive mode
- Include blast radius in every incident report
- For Tier-0 services, complete the blast radius assessment within the FIRST 5 minutes of triage