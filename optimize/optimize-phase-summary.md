# Optimize Phase — Cost & Value Improvement

**Project:** ha-web-portal · AWS us-east-1 · April 2026

---

## What This Phase Covers

The Optimize phase converts cost visibility from the Inform phase into actionable recommendations. Each optimization is evaluated across a value spectrum that extends beyond cost alone — encompassing reliability trade-offs, security posture, operational complexity, and long-term scalability. Recommendations are scoped to dev and prod phases separately, recognising that acceptable trade-offs differ materially between pre-production and live environments.

All analyses follow a consistent methodology: current state costs are bill-extrapolated from the 5-day actual billing period (actual ÷ 5 × 30). Where an alternative introduces a resource with no billing history, the base cost is calculated from first principles at 720 hours (April: 30 days × 24hrs); the usage cost is based on usage estimation for 500 employees within a similar time frame. Each analysis then models dev and prod costs separately across base and usage components, presents a trade-off matrix, and closes with a phased recommendation.

---

## Optimization Analyses

| Analysis | Monthly cost | Dev recommendation | Prod recommendation |
|---|---|---|---|
| [NAT Gateway](nat-gateway-optimization-analysis.md) | $27.30 (35%) | NAT Instance — saves $23.56/mo | NAT GW + VPC Endpoints — +$43.20/mo* |
| [VPC Public IPv4](vpc-public-ipv4-optimization-analysis.md) | $11.70 (15%) | Release NAT GW EIP — saves $3.90/mo | Current state |
| [EC2 Rightsizing](ec2-rightsizing-optimization-analysis.md) | $13.26 (17%) | Scheduled scaling — saves $2.72/mo | Scheduled scaling — saves $2.72/mo |
| [RDS](rds-optimization-analysis.md) | $12.00 (15%) | gp2 → gp3 — value optimization | gp3 + Reserved Instance — saves $2.88/mo |

*Cost increase — justified on security, operational resilience, and scalability grounds.

---

## Prioritization

Recommendations are scored on impact and effort using a 1–5 scale and plotted on the **FinOps Optimization Priority Matrix**. See Optimisation Roadmap: [optimize/finops-optimization-roadmap.md](https://github.com/Ola-raji/AWS-FinOps-Analysis--Internal-HR-Portal-Project/blob/main/optimize/finops-optimization-roadmap.md). [Interactive Optimization Priority Matrix](https://claude.ai/public/artifacts/8ad868dc-40dc-4c93-8d6f-50b5a7d9dbab)


**Scoring rubric**

| Dimension | Low | High |
|---|---|---|
| Impact | 1–2 | 3–5 |
| Effort | 1–2 | 3–5 |

| Quadrant | Impact | Effort | Decision rule |
|---|---|---|---|
| Prioritise | High (3–5) | Low (1–2) | Execute immediately — highest return per unit of effort |
| Invest | High (3–5) | High (3–5) | Plan and commit — strategic importance justifies the effort |
| Opportunistic | Low (1–2) | Low (1–2) | Execute when capacity allows — low cost, low return |
| Deprioritise | Low (1–2) | High (3–5) | Evaluated and consciously deprioritised |

The table below maps each optimization to its quadrant, reflecting where it falls on the impact-effort scoring rubric and the order in which it should be approached.

**Optimization inventory by quadrant**

| Quadrant | Items |
|---|---|
| Prioritise | EC2 scheduled scaling |
| Invest | NAT GW → NAT Instance · NAT GW + VPC Endpoints |
| Opportunistic | RDS gp2→gp3 · Release NAT GW EIP · RDS Reserved Instance |
| Deprioritise | Convert ALB to internal |

---

## Summary

| Phase | Gross baseline | Post-optimization | Net impact |
|---|---|---|---|
| Dev | $77.88/mo | ~$47.70/mo | −$30.18/mo |
| Prod | $80.13/mo | ~$42.53/mo | −$37.60/mo |

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
