# AWS Tagging Governance Policy

**Project:** ha-web-portal · AWS us-east-1  
**Technical owner:** Platform Team  
**Policy owner:** FinOps / Finance Team  
**Version:** 1.0 · April 2026

---

## Purpose

This policy defines tagging requirements, ownership, enforcement, and audit cadence for all AWS resources deployed under the `ha-web-portal` project. It exists to maintain cost allocation integrity and ensure Cost Explorer tag-based reporting remains accurate as the architecture evolves.

---

## Required Tags

All taggable AWS resources must carry the following six tags at deployment:

| Tag key | Value(s) | Notes |
|---|---|---|
| `Environment` | `production` | Fixed |
| `Project` | `ha-web-portal` | Fixed |
| `Owner` | `platform-team` | Fixed |
| `CostCenter` | `CC-1001` | Fixed |
| `Criticality` | `high` | Fixed |
| `Workload` | `web-tier` · `data-tier` · `network` · `observability` | Varies by resource |

Tag keys are case-sensitive. Values must match exactly — variations will not be recognised by Cost Explorer.

---

## Ownership Model

| Responsibility | Owner |
|---|---|
| Applying and maintaining tags | Platform Team |
| Weekly coverage audit | FinOps / Finance Team |
| Escalating non-compliance | FinOps / Finance Team |
| Activating tag keys in Billing console | FinOps / Finance Team |
| Policy updates | FinOps + Platform Team jointly |

---

## Enforcement

Compliance is enforced through **flagging** — resources deploy but non-compliant resources are surfaced in the weekly audit report. Blocking is deferred until tagging culture matures.

**Tooling:** AWS Config `required-tags` rule monitors all supported resource types. Non-compliant resources appear in the Config compliance dashboard and weekly audit report.

**Remediation window:** 5 business days from audit report date. Unresolved gaps are escalated by the FinOps Team.

**Known gap:** Auto-generated resources (e.g. ASG target tracking alarms) arrive untagged by design. These are identified in the weekly audit and manually remediated within the remediation window.

---

## Audit Cadence

| Activity | Frequency | Owner |
|---|---|---|
| Tag coverage audit — Tag Editor + AWS Config | Weekly — Monday | FinOps / Finance Team |
| Audit report distributed to Platform Team | Weekly — Monday | FinOps / Finance Team |
| Coverage trend review | Monthly | FinOps / Finance Team |
| Policy review | Quarterly | FinOps + Platform Team |

**Coverage rate target:** 95% of taggable resources carry all six tags at weekly audit. Below this threshold triggers an immediate remediation sprint.

---

## New Resource Procedure

1. Determine the correct `Workload` value for the resource's architectural tier
2. Apply all six tags at creation — not retroactively
3. Verify tags appear in Tag Editor within 24 hours
4. If the resource type is unsupported by Tag Editor, document it as an untaggable resource in the architecture record

> Tags applied retroactively do not backfill Cost Explorer data. Any allocation gap must be noted in the monthly coverage report with a proxy attribution applied.

---

*FinOps Portfolio Project · AWS us-east-1 · April 2026*
