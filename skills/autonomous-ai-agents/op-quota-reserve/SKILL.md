---
name: op-quota-reserve
description: Check a planned org fits the company monthly budget.
version: 1.0.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, budget, deterministic, ar-head]
    related_skills: [arh-org-skeleton, op-batch-create]
---

# op-quota-reserve Skill

Reads the company's monthly budget ceiling and what existing agents already commit, and compares that headroom against the planned org. This is two API reads and a subtraction, not a reasoning step — run it, do not reason about it.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 2 of an org bootstrap, before any design work
- Immediately before creation, to re-check after the design phase

Don't use for:

- Estimating headroom from an earlier reading — budget moves underneath you

## Prerequisites

- The company id
- Planned monthly budget per agent from the specs

## Never reach the Orchestrator's database directly

> This skill used to hand you a `psql` command — wrapped in a terminal tool call, run inside the database container as the owning role — and a query to go with it. **It is gone.** Reading the Orchestrator's schema is allowed; reaching it with a database client as the owning role is a different thing — that command carries no row-level security, no access check and no audit trail, it keeps working after a migration has moved the column underneath it, and being a shell it reaches whatever else it can reach. Read through the API.

## Quick Reference

```http
GET /api/companies/{companyId}
GET /api/companies/{companyId}/agents
```

From the company: `budgetMonthlyCents` (the ceiling), `status`, `requireBoardApprovalForNewAgents`.
From the agents array: sum `budgetMonthlyCents` where `status` is not `terminated` — that is committed.
Headroom is the ceiling minus committed. Field names are the API's camelCase, not table columns, and the agents route accepts **no** query parameters: it answers 400 to any of them.

## Procedure

1. Read ceiling, status and the approval gate with `GET /api/companies/{companyId}`.
   - Done when: The ceiling, the status and the approval flag are in hand from one response.
2. Read the agents with `GET /api/companies/{companyId}/agents` and sum the allowances of those not `terminated`.
   - Done when: Committed and headroom are computed from this run's response, and the agent count is recorded alongside them.
3. Compare planned spend against headroom.
   - Done when: The comparison uses the figures just read, not a cached value.
4. Abort on shortfall, reporting all four figures in dollars.
   - Done when: On shortfall nothing proceeds to design or creation.

## Pitfalls

- Computing headroom in the prompt instead of reading it. The Orchestrator is authoritative; a prompt is not.
- Checking once at design time only. Re-check before creation.
- Proceeding when requireBoardApprovalForNewAgents is true. That flag gates creation regardless of budget — surface it, never route around it.
- Provisioning into a company whose status is not active.
- **Trusting the sum without the count.** The agents list is filtered to what the calling actor may read, so a narrow actor sees fewer agents, sums less, and is told there is *more* headroom than there is — the one direction of error that lets provisioning through when it should stop. Run as an actor scoped to the whole company.
- Reading a missing `budgetMonthlyCents` as zero. Stop and report it instead.

## Verification

- [ ] All four figures were read in this run, not carried forward.
- [ ] Headroom is greater than or equal to planned spend.
- [ ] The board-approval flag is reported either way.
- [ ] The agent count is reported with the figures, so a filtered list is visible.
