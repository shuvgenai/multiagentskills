---
name: op-verify
description: Verify created agents against the Orchestrator.
version: 1.0.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, verification, deterministic, ar-head]
    related_skills: [op-batch-create, arh-handover-report]
---

# op-verify Skill

Counts and checks what was actually written, never what the creation log claims. Counting passes over one API response, not reasoning — a model asked to check its own work tends to agree with itself.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 10 of an org bootstrap, immediately after creation
- After a rollback, to confirm the undo was complete

Don't use for:

- Verifying from the creation log. The log records intent.

## Prerequisites

- The company id
- The plan, for comparison

## Never reach the Orchestrator's database directly

> This skill used to hand you a `psql` command — wrapped in a terminal tool call, run inside the database container as the owning role — and a recursive query to go with it. **It is gone.** Reading the Orchestrator's schema is allowed; reaching it with a database client as the owning role is a different thing — that command carries no row-level security, no access check and no audit trail, it keeps working after a migration has moved the column underneath it, and being a shell it reaches whatever else it can reach. Read through the API. The recursive CTE becomes a walk over `reportsTo` in memory: same rows, same answer.

## Quick Reference

```http
GET /api/companies/{companyId}/agents
GET /api/companies/{companyId}
```

The agents response is a plain array with `id`, `name`, `title`, `reportsTo`, `status`, `budgetMonthlyCents` and `metadata`. It accepts **no** query parameters and answers 400 to any of them, so every count below is a pass over that one array, keeping only agents whose `status` is not `terminated`:

- **total**, **roots** (`reportsTo === null`), **created by AR Head** (`metadata.created_by === "ar-head"`)
- **orphans** — every non-null `reportsTo` must name an `id` present in the same array; must be empty
- **depth profile** — from the roots at level 0, follow `reportsTo` downward and count per level
- **committed** — sum of `budgetMonthlyCents`, compared to the company's own `budgetMonthlyCents`

## Procedure

1. Read every agent with `GET /api/companies/{companyId}/agents`, then compute the counts and the orphan set from that array.
   - Done when: Orphan count is zero and the total is recorded.
2. Walk `reportsTo` from the roots and compare the depth profile against the plan.
   - Done when: Depth profile matches; any difference is reported per level.
3. Compare committed budget against the company ceiling from `GET /api/companies/{companyId}`.
   - Done when: Committed spend is at or below the ceiling.

## Pitfalls

- Verifying against the creation log rather than the Orchestrator.
- Treating an orphan as a warning. It is a failure; recommend rollback.
- Accepting more than one root without the Org Admin having asked for it.
- Ignoring a depth profile that differs from the plan — it means specs were dropped silently.
- **Diagnosing a scope problem as a broken org.** The agents list is filtered to what the calling actor may read: a missing parent reads as an orphan and a missing child as a short level, so a narrow actor produces exactly the defects this step hunts for. Run as an actor scoped to the whole company, report the total, and if it disagrees with the plan's count, say so and stop rather than rolling back rows that may be fine.

## Verification

- [ ] Orphan query returns zero rows.
- [ ] Depth profile matches the plan at every level.
- [ ] Committed budget is at or below the ceiling.
- [ ] Agent count equals planned count — and the count came from this run's response.
