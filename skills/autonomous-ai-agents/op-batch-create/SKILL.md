---
name: op-batch-create
description: Create planned agents in parent-before-child order.
version: 1.0.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, creation, deterministic, ar-head]
    related_skills: [op-validate-graph, op-verify]
---

# op-batch-create Skill

Creates the validated specs through the Orchestrator's agents API, depth by depth, parent before child. Every decision was made upstream; this step only executes. It is the only step that creates agents.

> **Never write the Orchestrator's tables directly.** This skill used to hand you a raw `INSERT` against the Orchestrator's `agents` table, and a `psql` command to run it with. Both are gone. `agents` is the Orchestrator's own table: you may read its schema, and you may write it only through its API. A direct insert bypasses every validator, every permission check and the approval gate, and the Orchestrator's next migration owes nothing to rows it did not create.

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

```http
POST /api/companies/{companyId}/agents
{
  "name": "...", "role": "...", "title": "...",
  "reportsTo": "<parent agent id, or null for the root>",
  "adapterType": "process", "adapterConfig": { ... },
  "budgetMonthlyCents": 0, "metadata": { ... }
}
```

`companyId` goes in the path, never the body. Field names are the API's camelCase, not table columns, and there is no `status` field — the Orchestrator owns an agent's initial state.

## Procedure

1. Sort specs by depth ascending.
   - Done when: The root is first.
2. Create them level by level, one API call per agent, keeping a key-to-id map so the next level resolves its parent from the returned id.
   - Done when: Every created agent has a `reportsTo` that already existed, except the root.
3. Record per-agent success or failure as you go, and stop at the first failure.
   - Done when: You can name exactly which planned agents now exist.

**There is no transaction across API calls, so there is no rollback.** Do not describe one. Hand a partial org to `arh-exception-triage` with the list of what exists.

## Pitfalls

- Creating a child before its parent. `agents_reports_to_agents_id_fk` refuses it outright, through the API as surely as under it; the sort is not optional.
- Calling with the wrong `companyId` in the path. The agent is created but invisible in that Org Admin's console.
- Trying to route around board approval where the Orchestrator requires it. It refuses the call; that is the gate working.
- Pushing on after a failure in the hope of a better total. Stop at the first one and hand to `arh-exception-triage`.

## Verification

- [ ] Created count equals planned count.
- [ ] Zero orphans immediately after the run.
- [ ] Every created agent sits under the correct company.
- [ ] On failure, the report names every agent that exists — no claim of a rollback the API cannot perform.
