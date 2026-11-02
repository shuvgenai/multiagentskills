---
name: arh-handover-report
description: Report what was created and what still needs an admin.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, reporting, handover, ar-head]
    related_skills: [op-verify, arh-exception-triage]
---

# arh-handover-report Skill

Summarises the verified result for the Org Admin and names everything still outstanding. Runs on Haiku: the facts are already computed, so there is no reasoning left to pay for.

## When to Use

- Final step of an org bootstrap, after `op-verify`
- After a rollback, to report what was undone

Don't use for:

- Reporting before verification — the creation log records intent, not outcome

## Prerequisites

- The verification output from `op-verify`
- Any credential gaps and pending skill approvals

## Procedure

1. Summarise created counts by division and band from the verification output.
   - Done when: Counts come from verification, not from the plan.
2. List every outstanding credential, approval and decision.
   - Done when: No outstanding item is omitted, including minor ones.
3. Name the three things to check first in the console.
   - Done when: Each named check is reachable with the Org Admin's own access.

## Pitfalls

- Reporting what was planned instead of what exists.
- Dropping minor outstanding items — they become the support call later.
- Proposing a next action that needs access the Org Admin does not have.

## Verification

- [ ] Every count traces to the verification output.
- [ ] Outstanding items are listed exhaustively.
- [ ] The next-action list is numbered and each item is actionable.
