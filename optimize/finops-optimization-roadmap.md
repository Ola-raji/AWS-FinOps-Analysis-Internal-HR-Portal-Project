# FinOps Optimization Roadmap

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal
**Baseline:** $77.88/month gross · April 2026

---

## Prioritization Framework

Each recommendation is scored on two dimensions using a 1–5 scale:

- **Impact** — value delivered across cost saving, security improvement, and operational resilience
- **Effort** — implementation complexity, risk, dependencies, and time required

Tier numbers reflect strategic priority weight. Implementation sequence follows execution readiness — Prioritise first, Invest and Opportunistic concurrently, Avoid last.

> *"Strategic investments are initiated concurrently with opportunistic actions rather than sequentially, ensuring high-effort, high-impact work is never deferred by lower-complexity tasks."*

### Scoring rubric

| Dimension | Low | High |
|---|---|---|
| Impact | 1–2 | 3–5 |
| Effort | 1–2 | 3–5 |

| Quadrant | Impact | Effort | Decision rule |
|---|---|---|---|
| Prioritise | High (3–5) | Low (1–2) | Execute immediately — highest return per unit of effort |
| Invest | High (3–5) | High (3–5) | Plan and commit — strategic importance justifies the effort |
| Opportunistic | Low (1–2) | Low (1–2) | Execute when capacity allows — low cost, low return |
| Deprioritise | Low (1–2) | High (3–5) | Deprioritise — low return relative to effort |

> Priority quadrant visualization: [FinOps Optimization Priority Matrix](finops-optimization-quadrant.html)

---

## Optimization Inventory

| # | Subject | Optimization | Phase | Impact | Effort | Quadrant |
|---|---|---|---|---|---|---|
| 1 | EC2 | Scheduled scaling — ASG off-hours standby | Dev + Prod | 4 | 1 | Prioritise |
| 2 | NAT Gateway | Replace NAT GW with NAT Instance (Alt B) | Dev | 3 | 4 | Invest |
| 3 | NAT Gateway | Retain NAT GW + VPC Endpoints (Alt C) | Prod | 4 | 4 | Invest |
| 4 | RDS | gp2 → gp3 storage migration | Dev + Prod | 2 | 1 | Opportunistic |
| 5 | VPC Public IPv4 | Release NAT GW EIP | Dev | 1 | 1 | Opportunistic |
| 6 | RDS | Reserved Instance — 1-year term | Prod | 2 | 2 | Opportunistic |
| 7 | VPC Public IPv4 | Convert ALB to internal scheme | Deferred | 1 | 5 | Deprioritise |

---

## Execution Sequence

### Dev phase

**Step 1 — Prioritise**
Execute high-impact, low-effort items immediately. Fast wins that build momentum with minimal friction.

- EC2 scheduled scaling — configure two ASG scheduled actions. Saving: **$2.72/month**

**Step 2 — Invest + Opportunistic (concurrent)**
Invest quadrant is entered simultaneously with Opportunistic. Opportunistic items close faster due to lower effort. Invest carries a longer runway but must not be deferred — strategic importance demands concurrent initiation.

- NAT GW → NAT Instance (Invest) — replace managed NAT Gateway with `t3.nano` EC2 NAT instance. Saving: **$23.56/month**
- RDS gp2 → gp3 (Opportunistic) — online migration, no downtime. Saving: **$0.00 — value optimization**
- Release NAT GW EIP (Opportunistic) — automatic byproduct of NAT Instance implementation. Saving: **$3.90/month**

---

### Prod phase

**Step 1 — Prioritise**
Carry forward any Prioritise items not completed in dev. No additional Prioritise items specific to prod.

- EC2 scheduled scaling (if not already implemented)

**Step 2 — Invest + Opportunistic (concurrent)**
Both quadrants entered simultaneously at go-live confirmation. Opportunistic closes faster. Invest carries a longer runway but must not be deferred.

- NAT GW + VPC Endpoints (Invest) — deploy Interface Endpoints for SSM and CloudWatch. Additional cost: **+$43.20/month** — justified on security, resilience, and scalability grounds
- RDS gp2 → gp3 (Opportunistic — carry forward if not done in dev)
- RDS Reserved Instance (Opportunistic) — purchase 1-year RI at go-live confirmation. Saving: **$2.88/month**

**Step 3 — Deprioritise**
Documented and consciously deprioritised. Revisited only when the network access strategy is reviewed.

- Convert ALB to internal — Client VPN prerequisite cost exceeds the $7.80/month saving at this scale

---

## Summary

### Dev phase

| Action | Quadrant | Monthly impact |
|---|---|---|
| EC2 scheduled scaling | Prioritise | +$2.72 |
| RDS gp2 → gp3 | Opportunistic | $0.00 (value opt.) |
| Release NAT GW EIP | Opportunistic | +$3.90 |
| NAT GW → NAT Instance | Invest | +$23.56 |
| **Total dev saving** | | **+$30.18/month** |

### Prod phase

| Action | Quadrant | Monthly impact |
|---|---|---|
| EC2 scheduled scaling | Prioritise | +$2.72 |
| RDS gp2 → gp3 | Opportunistic | $0.00 (value opt.) |
| RDS Reserved Instance | Opportunistic | +$2.88 |
| NAT GW + VPC Endpoints | Invest | -$43.20 (cost increase) |
| **Net prod impact** | | **-$37.60/month** |

> The negative prod impact reflects a deliberate decision to prioritise security posture and operational resilience over cost reduction. The NAT GW + VPC Endpoints recommendation increases cost but is justified on security, operational resilience, and scalability grounds — documented in full in the NAT Gateway Optimization Analysis.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
*Gross baseline: $77.88/month · Optimized dev target: ~$47.70/month*
