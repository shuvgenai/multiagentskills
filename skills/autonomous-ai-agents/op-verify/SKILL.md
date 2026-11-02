---
name: op-verify
description: Verify created agents against the database.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, verification, deterministic, ar-head]
    related_skills: [op-batch-create, arh-handover-report]
---

# op-verify Skill

Counts and checks what was actually written, never what the creation log claims. Count queries, not reasoning — a model asked to check its own work tends to agree with itself.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 10 of an org bootstrap, immediately after creation
- After a rollback, to confirm the undo was complete

Don't use for:

- Verifying from the creation log. The log records intent.

## Prerequisites

- The company id
- The plan, for comparison

## Quick Reference

```sql
-- orphans, must return zero rows
select a.id, a.name from agents a
 where a.company_id = :company_id and a.reports_to is not null
   and not exists (select 1 from agents p where p.id = a.reports_to);

-- depth profile
with recursive chain as (
  select id, reports_to, 0 as lvl from agents
   where company_id = :company_id and reports_to is null
  union all
  select a.id, a.reports_to, c.lvl+1 from agents a
    join chain c on a.reports_to = c.id where a.company_id = :company_id)
select lvl, count(*) from chain group by lvl order by lvl;
```

## Procedure

1. Count agents, then query for orphans, using terminal(command="docker exec orbitos-postgres-1 psql -U paperclip -d paperclip -c \"<query>\"").
   - Done when: Orphan count is zero.
2. Walk the hierarchy and compare the depth profile against the plan.
   - Done when: Depth profile matches; any difference is reported per level.
3. Compare committed budget against the company ceiling.
   - Done when: Committed spend is at or below the ceiling.

## Pitfalls

- Verifying against the creation log rather than the database.
- Treating an orphan as a warning. It is a failure; recommend rollback.
- Accepting more than one root without the Org Admin having asked for it.
- Ignoring a depth profile that differs from the plan — it means specs were dropped silently.

## Verification

- [ ] Orphan query returns zero rows.
- [ ] Depth profile matches the plan at every level.
- [ ] Committed budget is at or below the ceiling.
- [ ] Agent count equals planned count.
