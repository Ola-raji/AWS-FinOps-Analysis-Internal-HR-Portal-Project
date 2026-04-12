# RDS Optimization Analysis

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal
**Baseline:** $77.88/month gross · April 2026

---

## Current State

The RDS instance (`ha-project-db`) runs MySQL 8.0 on a `db.t3.micro` with 20GB of gp2 provisioned storage. The instance runs continuously — scheduling is not viable given the single-instance deployment and `Criticality = high` workload mandate. RDS charges are pure standing charges identical across dev and prod.

**Cost basis:** Bill-extrapolated from 5-day actual: RDS instance $1.68 × 6 = **$10.08/month** · storage $0.32 × 6 = **$1.92/month.** Alternative B introduces a new pricing tier — RI cost calculated at 720 hours (April-specific).

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| db.t3.micro instance-hours | $0.017/hr | Bill-extrapolated ($1.68 × 6) | Same |
| gp2 provisioned storage | $0.115/GB-month | Bill-extrapolated ($0.32 × 6) | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| db.t3.micro (on-demand) | $10.08 | $0.00 | $10.08 | $10.08 | $0.00 | $10.08 |
| gp2 storage (20GB) | $1.92 | $0.00 | $1.92 | $1.92 | $0.00 | $1.92 |
| **Total** | **$12.00** | **$0.00** | **$12.00** | **$12.00** | **$0.00** | **$12.00** |

---

## Alternative Analysis

**Alternative A — gp2 to gp3 storage migration**

Same cost as gp2. gp3 delivers 3,000 IOPS fixed baseline vs gp2's 60 IOPS at 20GB. Migration performed online by AWS — no downtime required. Instance cost unchanged — bill-extrapolated basis retained.

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| db.t3.micro instance-hours | $0.017/hr | Bill-extrapolated ($1.68 × 6) | Same |
| gp3 provisioned storage | $0.115/GB-month | Bill-extrapolated ($0.32 × 6) | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| db.t3.micro (on-demand) | $10.08 | $0.00 | $10.08 | $10.08 | $0.00 | $10.08 |
| gp3 storage (20GB) | $1.92 | $0.00 | $1.92 | $1.92 | $0.00 | $1.92 |
| **Total** | **$12.00** | **$0.00** | **$12.00** | **$12.00** | **$0.00** | **$12.00** |

Saving vs current: **$0.00/month.** Optimization is cost-to-value — same spend, 50× improvement in baseline IOPS.

---

**Alternative B — Reserved Instance, 1-year term**

1-year commitment reduces effective hourly rate by ~40%. New pricing tier — calculated at 720 hours ($0.010 × 720 = $7.20). Storage charges unaffected — bill-extrapolated basis retained.

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| db.t3.micro (1-year RI) | $0.010/hr (effective) | 720 hrs | 720 hrs |
| gp3 provisioned storage | $0.115/GB-month | Bill-extrapolated ($0.32 × 6) | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| db.t3.micro (1-year RI) | $7.20 | $0.00 | $7.20 | $7.20 | $0.00 | $7.20 |
| gp3 storage (20GB) | $1.92 | $0.00 | $1.92 | $1.92 | $0.00 | $1.92 |
| **Total** | **$9.12** | **$0.00** | **$9.12** | **$9.12** | **$0.00** | **$9.12** |

Saving vs current: **$2.88/month · $34.56/year (prod only).**

---

## Trade-Off Matrix

| | Current state | Alt A: gp2 → gp3 | Alt B: Reserved Instance |
|---|---|---|---|
| **Dev — base** | $12.00 | $12.00 | $9.12 |
| **Dev — usage** | $0.00 | $0.00 | $0.00 |
| **Dev — total** | $12.00 | $12.00 | $9.12 |
| **Prod — base** | $12.00 | $12.00 | $9.12 |
| **Prod — usage** | $0.00 | $0.00 | $0.00 |
| **Prod — total** | $12.00 | $12.00 | $9.12 |
| **Saving vs current (dev)** | — | $0.00 | +$2.88 |
| **Saving vs current (prod)** | — | $0.00 | **+$2.88** |
| Storage baseline IOPS | 60 | 3,000 | 3,000 |
| Storage throughput | Limited | 125 MB/s | 125 MB/s |
| Commitment required | None | None | 1-year term |
| Flexibility | Full | Full | Low — no cancellation |
| Downtime required | None | None | None |
| Production appropriate | Yes | Yes | Prod only |
| **Recommended phase** | — | **Dev + Prod** | **Prod** |

---

## Recommendation

**Immediate — Alternative A (dev and prod)**

Migrate from gp2 to gp3 immediately. Cost is unchanged but baseline IOPS improves from 60 to 3,000 — a value optimization with no downtime, no commitment, and no risk.

**Prod phase — Alternative B**

Purchase a 1-year Reserved Instance at go-live confirmation, reducing monthly instance cost from $10.08 to $7.20 — saving **$2.88/month ($34.56/year).** Combined with Alt A, total RDS monthly cost moves from $12.00 to **$9.12/month.**

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
*Gross baseline: $77.88/month · RDS share: 15%*
