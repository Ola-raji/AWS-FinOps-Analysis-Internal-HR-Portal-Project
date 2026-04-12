# EC2 Rightsizing & Compute Optimization Analysis

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal
**Baseline:** $77.88/month gross · April 2026

---

## Current State

Two `t3.micro` instances run continuously across private subnets in us-east-1a and us-east-1b, serving the web tier behind the ALB. The ASG maintains a minimum of 2 instances 24/7. EBS gp3 volumes persist regardless of instance state and are billed independently.

**Phase assumptions:**

EC2 instance charges are pure standing charges — no usage component applies. Dev and prod costs are identical at baseline. The prod phase introduces a scheduling opportunity: the HR portal serves 500 employees on business hours only (Monday–Friday, 9am–6pm), leaving approximately 525 hours/month where instances run with no active users.

**Cost basis:** Current state costs are bill-extrapolated from the 5-day actual billing period. EC2: $2.04 × 6 = **$12.24/month.** EBS: $0.17 × 6 = **$1.02/month.** Note: $0.0104 × 2 × 720hrs would yield $14.98 for EC2 — the extrapolated figure is used as it reflects actual observed spend. Alternative costs use 720 hours (April: 30 days × 24hrs) since scheduled scaling is a new configuration not present on the bill.

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| t3.micro instance-hours | $0.0104/hr per instance | Bill-extrapolated ($2.04 × 6) | Same |
| EBS gp3 storage | $0.08/GB-month | Bill-extrapolated ($0.17 × 6) | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| EC2 (2× t3.micro) | $12.24 | $0.00 | $12.24 | $12.24 | $0.00 | $12.24 |
| EBS (2× gp3) | $1.02 | $0.00 | $1.02 | $1.02 | $0.00 | $1.02 |
| **Total** | **$13.26** | **$0.00** | **$13.26** | **$13.26** | **$0.00** | **$13.26** |

---

## Alternative Analysis

**Alternative A — Scheduled scaling (business hours + 1 standby)**

ASG scheduled actions reduce capacity to 1 instance outside business hours (Monday–Friday, 9am–6pm). One standby instance is retained overnight and on weekends to handle any unplanned access and maintain a warm baseline. During business hours, ASG restores to minimum 2 instances as per current configuration.

New configuration — not present on bill. Costs calculated at 720 hours (April-specific): ~195 business hours + 525 off hours = 720 hours total.

| | Hours/month | Instances running |
|---|---|---|
| Business hours | ~195 hrs | 2 |
| Off hours | ~525 hrs | 1 |

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| t3.micro — business hours | $0.0104/hr per instance | 2 × 195 hrs | 2 × 195 hrs |
| t3.micro — off hours | $0.0104/hr per instance | 1 × 525 hrs | 1 × 525 hrs |
| EBS gp3 storage | $0.08/GB-month | Bill-extrapolated ($0.17 × 6) | Same |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| EC2 — business hours | $4.06 | $0.00 | $4.06 | $4.06 | $0.00 | $4.06 |
| EC2 — off hours (1 standby) | $5.46 | $0.00 | $5.46 | $5.46 | $0.00 | $5.46 |
| EBS (2× gp3) | $1.02 | $0.00 | $1.02 | $1.02 | $0.00 | $1.02 |
| **Total** | **$10.54** | **$0.00** | **$10.54** | **$10.54** | **$0.00** | **$10.54** |

Saving vs current: **$2.72/month (dev and prod).**

---

## Trade-Off Matrix

| | Current state | Alt A: Scheduled scaling |
|---|---|---|
| **Dev — base** | $13.26 | $10.54 |
| **Dev — usage** | $0.00 | $0.00 |
| **Dev — total** | $13.26 | $10.54 |
| **Prod — base** | $13.26 | $10.54 |
| **Prod — usage** | $0.00 | $0.00 |
| **Prod — total** | $13.26 | $10.54 |
| **Saving vs current (dev)** | — | **+$2.72** |
| **Saving vs current (prod)** | — | **+$2.72** |
| Min instances (business hours) | 2 | 2 |
| Min instances (off hours) | 2 | 1 |
| Off-hours resilience | Full HA | Reduced — single instance |
| Warm standby maintained | Yes | Yes — 1 instance |
| ASG scale-out preserved | Yes | Yes |
| Operational burden | Low | Low |
| Production appropriate | Yes | Yes |
| **Recommended phase** | — | **Dev + Prod** |

---

## Recommendation

**Dev and prod — Alternative A**

Scheduled scaling is recommended for both phases. Configuring ASG scheduled actions to reduce to 1 standby instance outside business hours reduces monthly EC2 cost from $13.26 to $10.54 — a saving of **$2.72/month (21%).** The single standby instance preserves warm capacity and handles any unplanned access without full cold-start latency. Full 2-instance HA is automatically restored at the start of each business day.

This is a native ASG capability requiring no additional tooling — implementation is two scheduled actions in the console. The trade-off is reduced overnight resilience, which is acceptable for an internal business-hours workload with no SLA obligations outside working hours.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
*Gross baseline: $77.88/month · EC2 + EBS share: 21%*
