---
name: arh-archetype-template
description: Write one reusable spec template per role archetype.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, templates, cost-control, ar-head]
    related_skills: [arh-division-populate, op-spec-expand]
---

# arh-archetype-template Skill

Groups roles into archetypes by band and division, then writes one reusable template per archetype rather than one spec per agent. This is where the cost of a large org is decided: 150 agents collapse to roughly 12 templates.

## When to Use

- Step 5 of an org bootstrap, after roles are listed
- A new band or division appears that no stored template covers

Don't use for:

- Writing a template per agent — that defeats the purpose of the step
- Re-templating an archetype already in the stored library

## Prerequisites

- The populated role list for every division
- The archetype library, if one exists from an earlier run

## Procedure

1. Group roles into archetypes keyed by band and division.
   - Done when: Every role maps to exactly one archetype.
2. For each archetype not already in the library, write one template with placeholders for name, title, parent and skills.
   - Done when: No template contains a specific agent name.
3. Return the templates keyed by archetype.
   - Done when: Template count is far below role count; for 150 agents expect 10-15.

## Pitfalls

- Writing one template per agent. If the count approaches the role count, the grouping is wrong.
- Baking a specific agent name into a template, which makes it unreusable.
- Re-writing an archetype the library already holds — reuse is what makes the second org cheaper than the first.

## Verification

- [ ] Template count is materially lower than role count.
- [ ] No template contains a literal agent name.
- [ ] Every archetype present in the role list has a template.
