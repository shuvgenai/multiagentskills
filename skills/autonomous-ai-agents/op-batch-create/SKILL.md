---
name: op-batch-create
description: Create planned agents in parent-before-child order.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, creation, deterministic, ar-head]
    related_skills: [op-validate-graph, op-verify]
---

# op-batch-create Skill

Inserts the validated specs into the agents table, depth by depth, inside a transaction. Every decision was made upstream; this step only executes. It is the only step that writes agents.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 9 of an org bootstrap, after validation passes and the Org Admin approves

Don't use for:

- Creating before validation passes
- Creating without a recorded approval

## Prerequisites

- Validated specs in creation order
- A recorded Org Admin approval
- A current budget check from `op-quota-reserve`

## Quick Reference

```sql
insert into agents
  (company_id, name, role, title, reports_to,
   adapter_type, adapter_config, budget_monthly_cents, metadata, status)
values (:company_id, :name, :role, :title, :parent_uuid,
        'process', :adapter_config::jsonb, :budget_monthly_cents,
        :metadata::jsonb, 'idle')
returning id;
```

## Procedure

1. Open a transaction and sort specs by depth ascending.
   - Done when: The root is first.
2. Insert level by level in batches, keeping a key-to-id map so the next level resolves its parent, using terminal(command="docker exec orbitos-postgres-1 psql -U paperclip -d paperclip -c \"<query>\"").
   - Done when: Every inserted row has a reports_to that already exists, except the root.
3. Record per-agent success or failure; commit only if the whole run succeeded.
   - Done when: Either every planned agent exists, or none does.

## Pitfalls

- Inserting a child before its parent. `agents_reports_to_agents_id_fk` refuses it outright; the sort is not optional.
- Omitting company_id. The agent is created but invisible in the console.
- Ignoring require_board_approval_for_new_agents.
- Pushing on after a batch fails more than one agent. A half-built org is worse than none — stop and hand to `arh-exception-triage`.

## Verification

- [ ] Created count equals planned count.
- [ ] Zero orphans immediately after commit.
- [ ] Every created row carries the correct company_id.
- [ ] On failure, the transaction rolled back and no partial org remains.
