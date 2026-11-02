---
name: arh-brief-intake
description: Turn a plain-English org brief into a blueprint.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, intake, org-design, ar-head]
    related_skills: [arh-org-skeleton, op-quota-reserve]
---

# arh-brief-intake Skill

Converts the Org Admin's free-text brief into a compact JSON blueprint of divisions and headcount. It does not design the hierarchy and does not name individual agents — that is `arh-org-skeleton`. Runs on Sonnet; the brief is short and reading it is comprehension, not deep reasoning.

## When to Use

- Step 1 of an org bootstrap, before any structure exists
- The Org Admin has described what the company does in prose

Don't use for:

- A structured blueprint already exists — go straight to `arh-org-skeleton`
- Adding one department to a live org — the brief is already scoped

## Prerequisites

- The Org Admin's brief text
- The company record, for industry and current budget

## Procedure

1. Read the brief for business lines, headcount and any named roles.
   - Done when: Every business line in the brief appears in the output.
2. Ask at most three clarifying questions, then proceed on stated assumptions.
   - Done when: No more than three questions asked; each unanswered item is listed as an assumption.
3. Emit JSON only: divisions, headcount per division, assumptions.
   - Done when: Output parses as JSON and contains no prose.

## Pitfalls

- Inventing headcount the brief neither states nor implies. If it is not there, list it as an assumption and let the Org Admin correct it.
- Emitting prose. Everything written here is re-read by every later step, so prose is paid for repeatedly.
- Proceeding on a brief that names no business function at all — stop and ask.

## Verification

- [ ] Output parses as JSON.
- [ ] Division count and total headcount are both present.
- [ ] Every assumption is stated explicitly rather than buried in a division name.
