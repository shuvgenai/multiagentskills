---
name: op-quota-reserve
description: Check a planned org fits the company monthly budget.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, budget, deterministic, ar-head]
    related_skills: [arh-org-skeleton, op-batch-create]
---

# op-quota-reserve Skill

Reads the company's monthly budget ceiling and what existing agents already commit, and compares that headroom against the planned org. This is a SQL query, not a reasoning step — run it, do not reason about it.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 2 of an org bootstrap, before any design work
- Immediately before creation, to re-check after the design phase

Don't use for:

- Estimating headroom from an earlier reading — budget moves underneath you

## Prerequisites

- The company id
- Planned monthly budget per agent from the specs

## Quick Reference

```sql
select c.budget_monthly_cents as ceiling_cents,
       coalesce(sum(a.budget_monthly_cents),0) as committed_cents,
       c.budget_monthly_cents - coalesce(sum(a.budget_monthly_cents),0) as headroom_cents,
       c.status, c.require_board_approval_for_new_agents
  from companies c
  left join agents a on a.company_id = c.id and a.status <> 'terminated'
 where c.id = :company_id group by c.id;
```

## Procedure

1. Read ceiling, committed and headroom with terminal(command="docker exec orbitos-postgres-1 psql -U paperclip -d paperclip -c \"<query>\"") using the query in Quick Reference.
   - Done when: Four figures are returned: ceiling, committed, headroom, status.
2. Compare planned spend against headroom.
   - Done when: The comparison uses the figures just read, not a cached value.
3. Abort on shortfall, reporting all four figures in dollars.
   - Done when: On shortfall nothing proceeds to design or creation.

## Pitfalls

- Computing headroom in the prompt instead of reading it. The database is authoritative; a prompt is not.
- Checking once at design time only. Re-check before creation.
- Proceeding when require_board_approval_for_new_agents is true. That flag gates creation regardless of budget — surface it, never route around it.
- Provisioning into a company whose status is not active.

## Verification

- [ ] All four figures were read in this run, not carried forward.
- [ ] Headroom is greater than or equal to planned spend.
- [ ] The board-approval flag is reported either way.
