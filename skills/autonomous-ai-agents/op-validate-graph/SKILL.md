---
name: op-validate-graph
description: Check an org plan for cycles, orphans and bad parents.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, validation, deterministic, ar-head]
    related_skills: [op-spec-expand, op-batch-create]
---

# op-validate-graph Skill

Proves the planned structure is sound before anything is written, and emits the creation order. Graph traversal, not reasoning — a model asked to check for cycles will sometimes say yes when the answer is no.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 8 of an org bootstrap, immediately before creation
- After any change to the planned structure

Don't use for:

- Validating a structure already created — that is `op-verify`

## Prerequisites

- The complete set of expanded specs

## Procedure

1. Walk every node upward to a root, detecting cycles.
   - Done when: Every node terminates at a root; no node is visited twice on one walk.
2. Check every parent exists and every skill and tool id resolves.
   - Done when: Zero unresolved references.
3. Sort by depth and emit the creation order.
   - Done when: The order places every parent before all of its children.

## Pitfalls

- Treating a problem as a warning. Any problem means nothing is created; there is no partial pass.
- Reporting problems by row number instead of agent key.
- Skipping the depth sort. `agents.reports_to` is a foreign key, so a child inserted first is refused by the database.

## Verification

- [ ] Zero cycles, zero orphans, zero unresolved references.
- [ ] Exactly one root.
- [ ] In the emitted order, every parent precedes its children.
