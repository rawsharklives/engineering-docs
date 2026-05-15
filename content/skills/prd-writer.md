---
title: "Skill: PRD Writer"
description: "Translates a feature brief into a structured Product Requirements Document."
summary: "Plays the role of Senior PM to produce a PRD with user stories, acceptance criteria, out-of-scope, open questions, and success metrics."
tags: ["skill", "product", "requirements"]
date: 2026-05-15
---

## What this skill does

The PRD Writer skill takes a feature brief — text, bullet points, or a voice transcript —
and produces a structured Product Requirements Document ready for system design and ticket
breakdown.

The AI plays the role of a Senior Product Manager. It does not suggest architecture or
implementation approaches — that is the [System Designer](../system-designer/) skill's
responsibility.

## When to use it

Use this skill at the start of any non-trivial feature. Skip it only for one-line bug
fixes or purely internal refactors with no user-visible behaviour change.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Feature brief | Yes | Text description of what the feature should do and why |
| Service page link | No | Name of the relevant service in engineering-docs |
| Relevant ADRs | No | Names or links of applicable architecture decisions |

## Outputs

A PRD containing:
- Feature summary (2–3 sentences)
- User stories (Given/When/Then, minimum 3)
- Acceptance criteria (numbered, testable, minimum 5)
- Out of scope (explicit list, minimum 3 items)
- Open questions (numbered list of ambiguities needing decisions)
- Success metrics (at least one quantitative metric)

## Stage gate

The skill stops after producing the PRD and tells the engineer to review before running
[System Designer](../system-designer/).

## How to invoke

**Claude Code:** `/skills/prd-writer` (requires `/setup-skills` to have been run first)

**Copilot (VS Code):** reference `.github/prompts/skills/prd-writer.prompt.md` in
Copilot Chat

**Canonical source:** `.claude/prompts/skills/prd-writer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [System Designer](../system-designer/) — next stage in the feature workflow
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/) — decision that introduced skills
- [Addendum C: Feature Workflow](../../adr/platform/addendum-c-feature-workflow/) — full workflow with stage gates
