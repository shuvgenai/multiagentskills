---
name: op-spec-expand
description: Expand archetype templates into one spec per agent.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, deterministic, specs, ar-head]
    related_skills: [arh-archetype-template, op-validate-graph]
---

# op-spec-expand Skill

Substitutes each role into its archetype template to produce a row-ready spec, derives tools from skills, and assembles the system prompt. A pure function: same input, same output, every time. This is the step that would otherwise cost one model call per agent.

> **Deterministic step.** Execute it; do not reason through it. If you find yourself deliberating here, the design has drifted and so has the cost.

## When to Use

- Step 7 of an org bootstrap, after templates exist

Don't use for:

- Generating prose per agent — substitution only

## Prerequisites

- Archetype templates
- The populated role list
- The tool registry

## Procedure

1. Select each role's archetype template by band and division.
   - Done when: Every role resolves to exactly one template.
2. Substitute name, title, parent, band, division and skills.
   - Done when: No placeholder remains unfilled in any spec.
3. Derive tools from skills and assemble the prompt. Carry parent key and depth as resolution hints.
   - Done when: Every spec carries a depth value, so creation order is already determined.

## Pitfalls

- Choosing tools. Tools follow from skills; choosing them lets an agent hold one no skill can use.
- Generating a prompt rather than substituting into the template.
- Emitting dollars where the schema expects monthly cents. 1500 means $15.00.
- Inventing an id. The database assigns it.

## Verification

- [ ] Spec count equals role count.
- [ ] No unfilled placeholders remain.
- [ ] Every spec has a depth value.
- [ ] Re-running produces byte-identical output.
