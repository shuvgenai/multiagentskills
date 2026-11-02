---
name: arh-skill-author
description: Author only the skills a plan needs but lacks.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, skill-authoring, ar-head]
    related_skills: [arh-division-populate, arh-archetype-template]
---

# arh-skill-author Skill

Diffs required skill ids against the catalog and authors only what is genuinely missing. The diff itself is set arithmetic and costs nothing; only authoring is paid for. Prefers reusing a near-match over creating a sibling.

## When to Use

- Step 6 of an org bootstrap, when roles reference skills that do not exist
- A new capability is needed that no catalog skill covers

Don't use for:

- Authoring a near-duplicate of an existing skill — extend it instead
- Authoring speculatively for roles that do not exist yet

## Prerequisites

- The full set of skill ids referenced by the role list
- The current catalog, read with `search_files` or `skill_view`

## Procedure

1. Diff required ids against the catalog. Do this arithmetically, not by reasoning.
   - Done when: The missing list contains only ids absent from the catalog.
2. For each missing id, decide: reuse an existing near-match, or author new.
   - Done when: Each missing id is resolved to reuse or author, with a reason.
3. Author the remainder and submit for Org Admin approval before anything depends on them.
   - Done when: No agent is created holding an unapproved skill.

## Pitfalls

- Authoring a near-duplicate. It costs twice: once now, and again every run that must choose between two similar skills.
- Naming a tool that is not in the registry.
- A missing count above roughly a quarter of required skills, which usually means the plan is wrong rather than the catalog incomplete — stop and say so.

## Verification

- [ ] Every previously missing id now resolves or is explicitly deferred.
- [ ] Every authored skill carries valid frontmatter.
- [ ] No authored skill duplicates an existing one.
