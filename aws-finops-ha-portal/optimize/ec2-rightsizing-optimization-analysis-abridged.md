# EC2 Rightsizing & Compute Optimization Analysis

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal
**Baseline:** $77.88/month gross · April 2026

---

## Current State

Two `t3.micro` instances run continuously across private subnets in us-east-1a and us-east-1b, maintained at a minimum of 2 by the ASG 24/7. EBS gp3 volumes are billed independently of instance state.

The HR portal serves 500 employees on business hours only (Monday–Friday, 9am–6pm), leaving ~525 hours/month where instances run with no active users. EC2 charges are pure standing charges — no usage component applies.

**Cost basis:** Bill-extrapolated from 5-day actual: EC2 $2.04 × 6 = **$12.24/month** · EBS $0.17 × 6 = **$1.02/month.** Alternative costs calculated at 720 hours (April-specific) as scheduled scaling is a new configuration not present on the bill.

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

ASG scheduled actions reduce capacity to 1 standby instance outside business hours. Full 2-instance configuration restored automatically at business day start. New configuration — calculated at 720 hours (~195 business hours + 525 off hours = 720 total).

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

Scheduled scaling reduces monthly EC2 cost from $13.26 to $10.54 — a saving of **$2.72/month (21%).** The single standby instance preserves warm capacity outside business hours. Implementation requires two ASG scheduled actions — no additional tooling needed. Reduced overnight resilience is an acceptable trade-off for a business-hours workload with no off-hours SLA.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
*Gross baseline: $77.88/month · EC2 + EBS share: 21%*
