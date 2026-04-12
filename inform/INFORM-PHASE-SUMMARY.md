# Inform Phase — Cost Visibility

**Project:** ha-web-portal · AWS us-east-1 · April 2026

---

## What This Phase Covers

The Inform phase establishes cost visibility across the architecture — answering the question: *what are we actually spending, on what, and why?* It is the foundation everything else builds on. Without accurate cost visibility, optimization recommendations are guesswork and governance has nothing to enforce.

This phase produced four outputs: a Cost Explorer analysis, a tagging schema and coverage audit, a cost allocation model, and a unit economics baseline.

---

## What Was Done

**Cost Explorer configuration**

AWS Cost Explorer was configured to expose gross usage charges. Three views were built and captured as evidence:

| View | Purpose | Screenshot |
|---|---|---|
| Service breakdown | Identify top cost drivers by service | [cost-explorer-service-breakdown.png](Ola-raji/AWS-FinOps-Analysis--Internal-HR-Portal-Project/inform/Cost Explorer Report/cost-explorer-by-service.png) |
| EC2-Other usage types | Expose NAT Gateway and public IPv4 charges hidden under EC2-Other | [cost-explorer-ec2-other.png](Ola-raji/AWS-FinOps-Analysis--Internal-HR-Portal-Project/inform/Cost Explorer Report/cost-explorer-ec2-other.png) |
| Daily spend trend | Establish burn rate baseline | [cost-explorer-daily-trend.png](Ola-raji/AWS-FinOps-Analysis--Internal-HR-Portal-Project/inform/Cost Explorer Report/cost-explorer-daily-trend.png) |

The EC2-Other view was the most revealing. NAT Gateway hourly charges completely dominated the bar, while data processing charges were negligible — confirming that NAT Gateway cost is almost entirely a fixed standing charge, not a traffic-driven one.

**Tagging and cost allocation**

A six-key tagging schema was designed and applied to all 16 taggable resources via AWS Tag Editor. Tags were subsequently activated in the Billing console as cost allocation dimensions.

**Cost baseline and unit economics**

A full-month cost projection was extrapolated from the 5-day actual billing period and used as the analytical baseline throughout this project.

---

## Gross Monthly Baseline — $77.88/month

Extrapolated from the 5-day actual billing period (April 1–5, 2026). Daily rate derived by dividing the 5-day actual by 5, then multiplied by 30 days (April).

| Service | 5-day actual | Daily rate (÷5) | Monthly projection (×30) | Cost share |
|---|---|---|---|---|
| NAT Gateway | $4.55 | $0.91 | $27.30 | 35.1% |
| ALB | $2.25 | $0.45 | $13.50 | 17.3% |
| EC2 + EBS | $2.21 | $0.44 | $13.26 | 17.0% |
| RDS | $2.00 | $0.40 | $12.00 | 15.4% |
| VPC Public IPv4 | $1.95 | $0.39 | $11.70 | 15.0% |
| S3 + CloudWatch | $0.02 | $0.004 | $0.12 | 0.2% |
| **Total** | **$12.98** | **$2.60** | **$77.88** | **100%** |

---

## Key Findings

| Finding | Detail |
|---|---|
| Gross monthly baseline | $77.88/month |
| Largest cost driver | NAT Gateway — $27.30/month (35%) |
| Network layer cost share | 67% of total spend (NAT Gateway + ALB + VPC Public IPv4) |
| Tagging coverage | 100% across 16 taggable resources |
| Cost per employee/month | $0.16 |
| Cost per working day | $3.54 |

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
