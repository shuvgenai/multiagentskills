---
name: arh-org-skeleton
description: Design divisions, heads and bands for an agent org.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, org-design, hierarchy, ar-head]
    related_skills: [arh-brief-intake, arh-division-populate]
---

# arh-org-skeleton Skill

Decides the top-level shape of the organization: which divisions exist, who heads each, what bands exist and how headcount splits across them. It does not name individual agents. This is the one step that runs on Opus, because every later step inherits its answer.

## When to Use

- Step 3 of an org bootstrap, after the blueprint and budget check
- A division needs restructuring and the shape must be redecided

Don't use for:

- Listing roles inside a division — that is `arh-division-populate`
- Writing specs — that is `arh-archetype-template` then `op-spec-expand`

## Prerequisites

- An approved blueprint from `arh-brief-intake`
- The span-of-control limit and the band definitions in use

## Procedure

1. Assign a head and a band to each division from the blueprint.
   - Done when: Every division has exactly one head.
2. Split headcount across divisions and insert manager layers so no manager exceeds the span-of-control limit.
   - Done when: No node exceeds the limit; added layers are justified by headcount, not taste.
3. Emit the skeleton: divisions, heads, bands, headcount, layer count, span targets.
   - Done when: Exactly one node has no parent; every other node names its parent.

## Pitfalls

- Exceeding span of control to avoid adding a layer. Add the layer.
- Producing more than one root. An org has one top node.
- Naming individual agents here. Roles come later; this step decides shape only.
- Treating this as cheap. It runs on Opus once because a wrong shape is the most expensive mistake in the run — everything downstream inherits it.

## Verification

- [ ] Exactly one node has no parent.
- [ ] Every leaf traces to the root by following parents.
- [ ] No node exceeds the span-of-control limit.
- [ ] Total headcount matches the approved blueprint.
