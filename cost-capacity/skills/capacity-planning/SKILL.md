---
name: capacity-planning
description: "Contoso's capacity governance policies: SKU approval matrix, scaling rules (scale-out preferred over scale-up), capacity thresholds, and node pool sizing guidelines. Use when recommending infrastructure scaling, evaluating resource utilization, or planning capacity changes."
---

# Capacity Planning — Contoso Governance

You are making capacity recommendations using Contoso's FinOps governance policies. These policies override generic Azure sizing recommendations.

## Contoso SKU Approval Matrix

| SKU Family | Example | Monthly Cost (per node) | Approval Required | Use Case |
|-----------|---------|------------------------|-------------------|----------|
| B-series (Burstable) | Standard_B4ms | ~$115 | Team lead | Dev/test, low-utilization workloads |
| D-series (≤ D4) | Standard_D4s_v5 | ~$280 | Team lead | Standard production workloads |
| D-series (> D4) | Standard_D8s_v5 | ~$560 | **VP Engineering** | High-compute workloads (requires justification) |
| D-series (> D16) | Standard_D16s_v5 | ~$1,120 | **CTO** | Exceptional cases only |
| E-series (Memory) | Standard_E4s_v5 | ~$320 | VP Engineering | Memory-intensive (databases, caches) |
| N-series (GPU) | Standard_NC6s_v3 | ~$2,200 | VP Engineering + FinOps | ML inference only |
| Spot instances | Any | ~60% discount | Team lead | Batch processing, fault-tolerant workloads |

### The Golden Rule: Scale-Out Before Scale-Up

**Contoso Policy CCP-001**: When a workload needs more capacity, first add more nodes of the SAME SKU (horizontal scaling). Only if horizontal doesn't work, request a larger SKU (vertical scaling).

## Safety Rules

- NEVER recommend SKU changes during active incidents — focus on scaling out
- ALWAYS show cost impact and approval requirements before any recommendation
- NEVER scale production pools below 2 nodes
- Flag any team forecasting > 90% budget consumption to the FinOps team