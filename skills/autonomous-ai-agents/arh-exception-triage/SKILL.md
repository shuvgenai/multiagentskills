---
name: arh-exception-triage
description: Classify a failed batch and recommend a next action.
version: 0.1.0
author: Shuv Chowdhury (shuvgenai), Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [provisioning, failure-handling, triage, ar-head]
    related_skills: [op-batch-create, arh-handover-report]
---

# arh-exception-triage Skill

Reads a failed creation batch, classifies the fault, and recommends continue, retry or roll back. Fires only on failure, which is why the happy path stays cheap.

## When to Use

- A creation batch failed
- Validation passed but creation did not

Don't use for:

- A clean run — this skill should not execute at all

## Prerequisites

- The failing batch's per-agent errors
- The ids created so far in this run

## Procedure

1. Read the per-agent errors from the failing batch.
   - Done when: Every failed agent is accounted for.
2. Classify the fault as transient, data, budget or design.
   - Done when: Exactly one classification, with the evidence that selected it.
3. Recommend one action with a reason.
   - Done when: The recommendation names the action and what it costs if wrong.

## Pitfalls

- Recommending continuation past a design fault. A bad structure does not improve by creating more of it.
- Retrying a budget fault without a new reservation.
- Guessing when the errors disagree with each other — escalate instead.

## Verification

- [ ] Exactly one classification is given.
- [ ] The recommendation is one of continue, retry or roll back.
- [ ] Every failed agent appears in the triage.
