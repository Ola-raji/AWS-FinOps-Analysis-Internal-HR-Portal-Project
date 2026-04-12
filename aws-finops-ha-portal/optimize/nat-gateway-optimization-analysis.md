# NAT Gateway Optimization Analysis

**Architecture:** HA Three-Tier AWS · us-east-1 · Internal HR Portal  
**Baseline:** $77.88/month gross · April 2026

---

## Current State

The NAT Gateway (`ha-nat-gateway`, us-east-1a) provides outbound internet access for EC2 instances in private subnets, serving SSM agent traffic, CloudWatch metrics shipping, and OS package updates. The single NAT Gateway deployment is intentional.

**Phase assumptions and pricing basis:**

Two phases are modelled. In the **dev phase**, the architecture is pre-production with near-zero outbound traffic — data processing charges are negligible and cost is driven almost entirely by the fixed hourly standing charge. In the **prod phase** (projected one month from now), the HR portal serves 500 employees. Application traffic, identity provider calls, payroll integrations, and CloudWatch log shipping are projected to generate approximately 50GB/month of outbound data.

**Cost basis:** Current state base cost is bill-extrapolated from the 5-day actual billing period: $4.55 (actual) × 6 = **$27.30/month.** Note: $0.045 × 720hrs would yield $32.40 — the extrapolated figure is used throughout as it reflects actual observed spend. Alternatives are new resources not present on the bill — costs calculated at 720 hours (April: 30 days × 24hrs).

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| NatGateway-Hours | $0.045/hr | Bill-extrapolated ($4.55 × 6) | Same |
| NatGateway-Bytes | $0.045/GB | ~0.07 GB | ~50 GB |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| NatGateway-Hours | $27.30 | — | — | $27.30 | — | — |
| NatGateway-Bytes | — | $0.00 | — | — | $2.25 | — |
| **Total** | **$27.30** | **$0.00** | **$27.30** | **$27.30** | **$2.25** | **$29.55** |

---

## Alternative Analysis

### Alternative A — VPC Interface Endpoints only

Five Interface Endpoints deployed across 2 AZs for SSM, CloudWatch, and EC2 API services. NAT Gateway is removed entirely. No internet egress capability — package updates and third-party API calls are blocked. New resource — cost calculated at 720 hours.

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| Endpoint-hours | $0.01/hr per endpoint per AZ | 5 endpoints × 2 AZs × 720 hrs | Same |
| Endpoint data processed | $0.01/GB | ~0 GB | ~5 GB (AWS service traffic only) |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| Endpoints (5 × 2 AZs) | $72.00 | $0.00 | $72.00 | $72.00 | $0.05 | $72.05 |
| NAT Gateway | None | None | None | None | None | None |
| **Total** | **$72.00** | **$0.00** | **$72.00** | **$72.00** | **$0.05** | **$72.05** |

Cost more than doubles vs current state and removes internet egress. Not recommended.

---

### Alternative B — NAT Instance (t3.nano)

Managed NAT Gateway replaced with a self-operated EC2 instance. No per-GB processing charge applies — the instance handles NAT at the OS level. At 50GB/month prod volume, data transfer out remains within AWS free tier thresholds. New resource — cost calculated at 720 hours ($0.0052 × 720 = $3.74).

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| t3.nano instance-hours | $0.0052/hr | 720 hrs | 720 hrs |
| Data transfer out | $0.09/GB (after free tier) | ~0.07 GB | ~50 GB (within free tier) |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| t3.nano instance | $3.74 | $0.00 | $3.74 | $3.74 | $0.00 | $3.74 |
| **Total** | **$3.74** | **$0.00** | **$3.74** | **$3.74** | **$0.00** | **$3.74** |

Saving vs current state: **$23.56/month (dev) · $25.81/month (prod)**. No managed failover, high operational burden, 128Mbps bandwidth ceiling.

---

### Alternative C — Retain NAT Gateway + Selective VPC Endpoints

Managed NAT Gateway retained for all internet-bound egress. VPC Endpoints added for SSM and CloudWatch only, redirecting AWS-internal traffic off the public internet path. NAT Gateway data processing charges reduced as AWS service traffic no longer flows through it.

**Cost basis:** NAT Gateway base cost uses bill-extrapolated figure ($27.30) — same resource as current state. VPC Endpoints are new resources not present on the bill — costs calculated at 720 hours (3 endpoints × 2 AZs × $0.01 × 720hrs = $43.20).

| Charge | Rate | Dev expected usage | Prod expected usage |
|---|---|---|---|
| NAT Gateway-Hours | $0.045/hr | Bill-extrapolated ($4.55 × 6) | Same |
| NAT Gateway-Bytes | $0.045/GB | ~0.07 GB | ~45 GB (internet only, AWS traffic redirected) |
| SSM Endpoint-hours | $0.01/hr per endpoint per AZ | 3 endpoints × 2 AZs × 720 hrs | Same |
| Endpoint data processed | $0.01/GB | ~0 GB | ~5 GB |

| | Dev — Base | Dev — Usage | Dev — Total | Prod — Base | Prod — Usage | Prod — Total |
|---|---|---|---|---|---|---|
| NAT Gateway | $27.30 | $0.00 | $27.30 | $27.30 | $2.03 | $29.33 |
| SSM Endpoints (3 × 2 AZs) | $43.20 | $0.00 | $43.20 | $43.20 | $0.05 | $43.25 |
| **Total** | **$70.50** | **$0.00** | **$70.50** | **$70.50** | **$2.08** | **$72.58** |

Cost increase vs current: **+$43.20 (dev) · +$43.03 (prod)**. See recommendation below for justification.

---

## Trade-Off Matrix

| | Current state | Alt A: Endpoints only | Alt B: NAT Instance | Alt C: NAT GW + Endpoints |
|---|---|---|---|---|
| **Dev — base** | $27.30 | $72.00 | $3.74 | $70.50 |
| **Dev — usage** | $0.00 | $0.00 | $0.00 | $0.00 |
| **Dev — total** | $27.30 | $72.00 | $3.74 | $70.50 |
| **Prod — base** | $27.30 | $72.00 | $3.74 | $70.50 |
| **Prod — usage** | $2.25 | $0.05 | $0.00 | $2.08 |
| **Prod — total** | $29.55 | $72.05 | $3.74 | $72.58 |
| **Saving vs current (dev)** | — | -$44.70 | **+$23.56** | -$43.20 |
| **Saving vs current (prod)** | — | -$42.50 | **+$25.81** | -$43.03 |
| Managed reliability | Yes | Yes | No | Yes |
| AWS service traffic | Public internet | Private | Public internet | Private |
| SSM independent of NAT | No | Yes | No | Yes |
| **Internet egress** | **Yes** | **No — NAT GW removed** | **Yes** | **Yes** |
| Operational burden | Low | Low | High | Low |
| Production appropriate | Yes | No | No | Yes |
| **Recommended phase** | — | — | **Dev** | **Prod** |

---

## Recommendation

**Short term — Alternative B (dev phase)**

During development, traffic is minimal and the workload is not yet business-critical. Replacing the NAT Gateway with a `t3.nano` NAT instance reduces monthly cost from $27.30 to $3.74 — a saving of **$23.56/month**. The reliability and operational trade-offs are acceptable pre-production.

**Production phase — Alternative C**

Alternative C costs more than the current state in prod ($72.58 vs $29.55). The recommendation is not made on cost grounds — it is made on three grounds that matter for a live HR portal:

First, **security.** All SSM and CloudWatch traffic moves off the public internet onto AWS private network paths, which is the appropriate posture for a workload handling employee payroll and personal data.

Second, **operational resilience.** VPC Endpoints decouple the operations path from the application path. If the NAT Gateway degrades, SSM access to instances remains intact — preserving the ability to diagnose and remediate incidents without dependency on the component under failure.

Third, **scalability.** At 50GB/month, Alt C does not save money. As the user base grows beyond 500 employees and outbound traffic scales, NAT Gateway data processing charges compound at $0.045/GB while endpoint costs remain fixed. The cost crossover point is approximately 975GB/month of AWS-internal traffic redirected through endpoints. Deploying endpoints now positions the architecture to benefit from that crossover as the product scales.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*  
*Gross baseline: $77.88/month · NAT Gateway share: 35%*
