# AWS FinOps Analysis — Internal HR Portal Project 

---

### What is this project about

This project takes a previously built production-grade, highly available three-tier AWS architecture and applies end-to-end FinOps practice to it — using it as the infrastructure subject for a real cost visibility, optimization, and governance exercise.

The architecture was originally designed as a general-purpose HA pattern, subsequently adopted as the foundation for an internal read-only HR portal serving 500 employees. The portal provides staff access to employee records, payroll information, HR policies, benefits documentation, and the organisational directory — all read-only, with no write or transactional capability.

All cost analysis is grounded in real AWS billing data, with usage projections modelled against a 500-employee workload operating standard business hours — Monday to Friday, 9am to 5pm.

---

### Assumptions

The existing architecture's capacity is assumed sufficient to serve 500 employees for the purposes of this exercise, operating within AWS Free Tier and promotional credit constraints. The architecture's core design principles — high availability, scalability, fault tolerance, reliability, and operational simplicity — are left fully intact across all phases of this analysis.

---

### Inform — Cost Visibility

> Full report: [inform-phase-summary.md](https://github.com/Ola-raji/AWS-FinOps-Analysis--Internal-HR-Portal-Project/blob/main/inform/inform-phase-summary.md)

Real billing data extracted from AWS Cost Explorer and analysed against the workload scenario. Key gap identified on first pass: AWS Cost Explorer defaults to net costs after credits — all charges appeared as $0.00 until a Charge Type = Usage filter was applied. All figures in this portfolio reflect gross spend.

**Gross monthly baseline — $77.88/month**

| Service | Monthly cost | Share |
|---|---|---|
| NAT Gateway | $27.30 | 35.1% |
| ALB | $13.50 | 17.3% |
| EC2 + EBS | $12.24 | 17.0% |
| RDS | $12.00 | 15.4% |
| VPC Public IPv4 | $11.70 | 15.0% |
| S3 + CloudWatch | $0.12 | 0.2% |

**Key findings:**
- NAT Gateway is the single largest cost driver at 35% — almost entirely a fixed standing charge with near-zero data processing at current traffic volumes
- 67% of total spend sits on the network layer (NAT Gateway + ALB + VPC Public IPv4) — not on compute or database
- AWS introduced a $0.005/hr public IPv4 charge in February 2024 — three continuously running IPs add $11.70/month with no usage component
- 16 taggable resources identified and tagged across 6 cost allocation dimensions. 100% tagging coverage achieved

| | |
|---|---|
| Cost per employee/month | $0.16 |
| Cost per working day | $3.54 |
| Networking cost ratio | 50% |

---

### Optimize — Cost & Value Improvement

> Roadmap: [finops-optimization-roadmap.md](optimize/finops-optimization-roadmap.md) · Interactive quadrant: [assets/finops-optimization-quadrant.html](assets/finops-optimization-quadrant.html)

Four optimization analysis were produced, covering major cost drivers. Each follows a consistent structure: current state costs are bill-extrapolated from the 5-day actual billing period (actual ÷ 5 × 30). Where an alternative introduces a resource with no billing history, the base cost is calculated from first principles by multiplying the cost per hour by 720 hours (April: 30 days × 24 hours); the usage cost is based on usage estimation for 500 employees within a similar time frame. Each analysis then models dev and prod costs separately across base and usage components, presents a trade-off matrix, and closes with a phased recommendation.

**Optimization inventory**

| # | Optimization | Analysis | Quadrant | Dev impact | Prod impact |
|---|---|---|---|---|---|
| 1 | EC2 scheduled scaling | [↗](optimize/ec2-rightsizing-optimization-analysis.md) | Prioritise | +$2.72/mo | +$2.72/mo |
| 2 | NAT GW → NAT Instance | [↗](optimize/nat-gateway-optimization-analysis.md) | Invest | +$23.56/mo | — |
| 3 | NAT GW + VPC Endpoints | [↗](optimize/nat-gateway-optimization-analysis.md) | Invest | — | +$43.20/mo* |
| 4 | RDS gp2 → gp3 | [↗](optimize/rds-optimization-analysis.md) | Opportunistic | Value opt. | Value opt. |
| 5 | Release NAT GW EIP | [↗](optimize/vpc-public-ipv4-optimization-analysis.md) | Opportunistic | +$3.90/mo | — |
| 6 | RDS Reserved Instance | [↗](optimize/rds-optimization-analysis.md) | Opportunistic | — | +$2.88/mo |
| 7 | Convert ALB to internal | [↗](optimize/vpc-public-ipv4-optimization-analysis.md) | Deprioritise | Deferred | Deferred |

*Cost increase — justified on security, operational resilience, and scalability grounds. See [NAT Gateway analysis](optimize/nat-gateway-optimization-analysis.md).

**Summary**

| Phase | Gross baseline | Post-optimization | Impact |
|---|---|---|---|
| Dev | $77.88/mo | ~$47.70/mo | -$30.18/mo saving |
| Prod | $80.13/mo | ~$42.53/mo | -$37.60/mo net* |

*Prod net impact reflects the NAT GW + VPC Endpoints cost increase absorbed against savings elsewhere.

---

### Operate — Cost Governance

> Policy: [tagging-governance-policy.md](operate/tagging-governance-policy.md) · Budget setup: [budget-and-anomaly-detection.md](operate/budget-and-anomaly-detection.md)

A governance model established to protect cost posture over time — preventing optimization gains from eroding as the architecture evolves.

**Budget configuration**

| Control | Configuration |
|---|---|
| AWS Budgets threshold | $47.70/month (optimized dev target) |
| Early warning alert | 80% — $38.16 |
| Breach alert | 100% — $47.70 |
| Notification channel | SNS → ha-project-alerts |

**Cost Anomaly Detection**

| Setting | Value |
|---|---|
| Monitor scope | Tag: Project = ha-web-portal |
| Alert threshold | $5.00 impact (~2× daily baseline) |
| Alert frequency | Daily summary digest |

**Tagging governance**

| Dimension | Detail |
|---|---|
| Tag keys | Environment · Project · Owner · CostCenter · Criticality · Workload |
| Coverage at inception | 100% — 16 of 16 taggable resources |
| Coverage target | 95% at weekly audit |
| Audit owner | FinOps / Finance Team — every Monday |
| Enforcement | Flag-based via AWS Config `required-tags` rule |

---

*AWS us-east-1 · April 2026 · Gross baseline: $77.88/month*
