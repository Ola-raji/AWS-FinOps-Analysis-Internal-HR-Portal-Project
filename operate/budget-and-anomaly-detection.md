# Budget & Anomaly Detection

**Project:** ha-web-portal · AWS us-east-1 · April 2026

---

## AWS Budgets

A monthly cost budget was configured scoped to the `ha-web-portal` project tag, set at the post-optimization dev target rather than the gross baseline — holding spend to the optimized state rather than the unoptimized one.

| Setting | Value |
|---|---|
| Budget name | ha-web-portal-monthly |
| Budget type | Cost budget |
| Period | Monthly |
| Threshold | $47.70 |
| Scope | Tag: Project = ha-web-portal |

**Alert thresholds**

| Alert | Threshold | Amount | Channel |
|---|---|---|---|
| Early warning | 80% actual spend | $38.16 | SNS → ha-project-alerts |
| Breach | 100% actual spend | $47.70 | SNS → ha-project-alerts |

---

## Cost Anomaly Detection

A Cost Anomaly Detection monitor was configured to catch unexpected spend patterns that fall below the monthly budget threshold — single-day spikes, gradual drift, or unplanned resource charges that aggregate slowly.

| Setting | Value |
|---|---|
| Monitor name | ha-web-portal-monitor |
| Monitor type | Cost category — Tag: Project = ha-web-portal |
| Alert threshold | $5.00 individual impact |
| Alert frequency | Daily summary digest |
| Notification | Email |

The $5.00 threshold represents approximately 2× the daily baseline ($80.13 ÷ 30 = $2.67/day) — meaningful enough to investigate, high enough to avoid alert fatigue. Daily summary was chosen over individual alerts to consolidate notifications and prevent desensitisation.

**Most likely anomaly triggers for this architecture**

| Scenario | Signal |
|---|---|
| NAT Gateway data spike | Unexpected outbound traffic — misconfigured app or external API call volume increase |
| EC2 scaling event not scaling back | ASG scales out but scale-in cooldown stalls — extra instances running beyond business hours |
| RDS storage autoscale | RDS storage grows past 20GB and autoscales — permanent cost increase, cannot be reversed |

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
