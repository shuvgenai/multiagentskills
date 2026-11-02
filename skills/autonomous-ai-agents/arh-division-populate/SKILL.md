---
name: arh-division-populate
description: List one division's roles with the skills each needs.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, roles, skills, ar-head]
    related_skills: [arh-org-skeleton, arh-skill-author]
---

# arh-division-populate Skill

Expands one division of the skeleton into named roles, each with its band, parent and required skills. Runs once per division rather than once per agent, and the skill catalog is identical across those calls so it caches.

## When to Use

- Step 4 of an org bootstrap, once per division in the approved skeleton
- A single division needs its roles re-listed after a structure change

Don't use for:

- Populating several divisions in one call — one call per division keeps the cached prefix stable
- Choosing tools — tools are derived from skills by `op-spec-expand`

## Prerequisites

- The approved skeleton entry for this division
- The current skill catalog, read with `search_files` or `skill_view`

## Procedure

1. Read the skeleton entry for this division only.
   - Done when: No role outside this division appears in the output.
2. Name each role with its band, its parent, and the skills it requires.
   - Done when: Every role names a parent that exists in the skeleton.
3. Use catalog skill ids verbatim; list anything missing rather than inventing it.
   - Done when: Every skill id either resolves in the catalog or appears in the missing list.

## Pitfalls

- Inventing a skill id that looks plausible. A fabricated id fails at validation and costs a whole re-run.
- Returning tools or prompts. Those are derived downstream.
- Explaining the reasoning. Only the role list is consumed.

## Verification

- [ ] Every role names an existing parent.
- [ ] Every skill id resolves in the catalog or is listed as missing.
- [ ] Role count matches the headcount the skeleton allocated to this division.
