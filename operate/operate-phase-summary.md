# Operate Phase — Cost Governance

**Project:** ha-web-portal · AWS us-east-1 · April 2026

---

## What This Phase Covers

The Operate phase establishes the governance layer that protects cost posture over time. Optimization gains are only durable if the conditions that produced them are maintained — tagging coverage doesn't erode, spend doesn't drift past the optimized baseline, and unexpected cost anomalies are caught before they compound.

Three controls were put in place: a monthly budget with tiered alerting, automated anomaly detection, and a tagging governance policy with a defined ownership model and audit cadence.

---

## Controls Summary

| Control | Tool | Configuration | Document |
|---|---|---|---|
| Monthly budget | AWS Budgets | $47.70 threshold · 80% early warning | [budget-and-anomaly-detection.md](budget-and-anomaly-detection.md) |
| Anomaly detection | Cost Anomaly Detection | $5.00 impact · daily digest | [budget-and-anomaly-detection.md](budget-and-anomaly-detection.md) |
| Tagging governance | AWS Config + Tag Editor | Weekly audit · 95% coverage target | [tagging-governance-policy.md](tagging-governance-policy.md) |

---

## Governance Rationale

**Budget** — sets a hard reference point. Any spend trending toward or past the optimized target ($47.70/month) triggers an alert before it becomes a problem. The 80% early warning threshold gives time to investigate before the ceiling is breached.

**Anomaly detection** — catches what budgets miss. A single-day NAT Gateway data spike or an unexpected RDS storage autoscale event won't move the monthly budget needle enough to trigger an alert, but it signals something worth investigating. The $5.00 threshold represents approximately 2× the daily baseline — meaningful enough to act on, high enough to avoid noise.

**Tagging governance** — protects cost allocation integrity. Tags are the foundation of every Cost Explorer grouping in this project. Without active governance, coverage erodes as new resources are deployed — and untagged spend becomes unallocated spend, undermining the entire Inform phase output.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
