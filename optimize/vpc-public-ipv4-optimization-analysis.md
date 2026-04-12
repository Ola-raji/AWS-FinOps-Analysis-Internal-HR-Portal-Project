# VPC Public IPv4 Optimization Analysis

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal
**Baseline:** $77.88/month gross · April 2026

---

## Current State

AWS introduced a charge of $0.005/hr per in-use public IPv4 address in February 2024. This architecture carries three public IPs continuously — one Elastic IP on the NAT Gateway and one auto-assigned IP per ALB node across both AZs.

Unlike NAT Gateway, this charge has no usage component. Dev and prod costs are identical — traffic volume has no bearing on public IP charges.

**Cost basis:** Bill-extrapolated from 5-day actual period. The bill recorded 390 IP-hours × $0.005 = $1.95 over 5 days. Extrapolated to 30 days: $1.95 × 6 = $11.70 total · $11.70 ÷ 3 IPs = **$3.90 per IP per month.**

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| Public IPv4 — NAT GW EIP | $0.005/hr | 3.25 IPs × 5 days extrapolated | Same |
| Public IPv4 — ALB node 1a | $0.005/hr | Same | Same |
| Public IPv4 — ALB node 1b | $0.005/hr | Same | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| NAT GW EIP | $3.90 | $0.00 | $3.90 | $3.90 | $0.00 | $3.90 |
| ALB node 1a | $3.90 | $0.00 | $3.90 | $3.90 | $0.00 | $3.90 |
| ALB node 1b | $3.90 | $0.00 | $3.90 | $3.90 | $0.00 | $3.90 |
| **Total** | **$11.70** | **$0.00** | **$11.70** | **$11.70** | **$0.00** | **$11.70** |

**Architectural note:** The ALB is internet-facing — a design decision made before the internal HR portal use case was defined. Converting to an internal ALB would eliminate the two ALB IP charges but requires a Client VPN or Direct Connect solution for employee access, costing significantly more than the $7.80/month saving at production scale. Deferred to a future network access strategy review.

---

## Alternative Analysis

**Alternative A — Release NAT Gateway EIP**

NAT Gateway EIP released as a direct byproduct of implementing the NAT Gateway Alternative B recommendation. No additional work required. Two ALB IPs remain — cost derived using the same bill-extrapolated rate of $3.90/IP/month.

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| Public IPv4 — NAT GW EIP | $0.005/hr | None (released) | None (released) |
| Public IPv4 — ALB nodes | $0.005/hr | 2 IPs × bill-extrapolated rate | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| NAT GW EIP | $0.00 | $0.00 | $0.00 | $0.00 | $0.00 | $0.00 |
| ALB node 1a | $3.90 | $0.00 | $3.90 | $3.90 | $0.00 | $3.90 |
| ALB node 1b | $3.90 | $0.00 | $3.90 | $3.90 | $0.00 | $3.90 |
| **Total** | **$7.80** | **$0.00** | **$7.80** | **$7.80** | **$0.00** | **$7.80** |

---

## Trade-Off Matrix

| | Current state | Alt A: Release NAT GW EIP |
|---|---|---|
| **Dev — base** | $11.70 | $7.80 |
| **Dev — usage** | $0.00 | $0.00 |
| **Dev — total** | $11.70 | $7.80 |
| **Prod — base** | $11.70 | $7.80 |
| **Prod — usage** | $0.00 | $0.00 |
| **Prod — total** | $11.70 | $7.80 |
| **Saving vs current (dev)** | — | **+$3.90** |
| **Saving vs current (prod)** | — | +$3.90 |
| Internet egress | Yes | Yes |
| Employee access | Public internet | Public internet |
| Operational burden | Low | Low |
| Additional dependencies | None | NAT GW Alt B implemented |
| Production appropriate | Yes | Dev only |
| **Recommended phase** | **Prod** | **Dev** |

---

## Recommendation

**Dev phase — Alternative A**

Releasing the NAT Gateway EIP is a zero-effort saving — it occurs automatically when the NAT Gateway is replaced with a NAT instance as recommended in the NAT Gateway analysis. Saving: **$3.90/month.**

**Prod phase — Current state**

No cost-effective optimization exists for the ALB public IP charges at this scale. The $11.70/month public IPv4 charge is accepted as the least-cost available option and documented as a known architectural misalignment for future remediation.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
*Gross baseline: $77.88/month · VPC Public IPv4 share: 14%*
