---
title: "Skill: Retrospective"
description: "Facilitates a blameless written retrospective for a completed feature."
summary: "Plays the role of Engineering Lead to produce a structured retro document with planned vs shipped comparison, what went well/poorly, and action items with named owners."
tags: ["skill", "retrospective", "process"]
date: 2026-05-15
---

## What this skill does

The Retrospective skill takes the original feature brief, the PRD, the ADR, and the Issue
delivery data, then produces a structured blameless retrospective document. It raises the
document as a PR to engineering-docs.

The AI plays the role of an Engineering Lead facilitating a written retro. The document
focuses on process, systems, and tooling — not individuals.

## When to use it

After documentation PRs have been merged — at the close of the feature workflow. This is
the final stage gate.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Original feature brief | Yes | What was originally asked for |
| Merged PRD | Yes | The scoped acceptance criteria |
| Merged ADR | Yes | The architecture as designed |
| Issue list with sizing | Yes | Which Issues closed and whether they came in on estimate |
| QA gap analysis | No | P0/P1 gaps found during QA, to feed into "what went poorly" |
| Anything else notable | No | Surprises, scope changes, incidents |

## Outputs

A retro document containing:
- Planned vs shipped scope comparison
- Timeline and estimation accuracy
- What went well (minimum 3 specific items)
- What went poorly (minimum 3 specific items)
- Action items with named owners and due dates (minimum 3)
- Lessons learned

Raised as a PR to `content/rca/` in engineering-docs.

## Stage gate

After raising the PR, the skill stops and tells the engineer to review as a team and
create GitHub Issues for each action item. This is the final stage in the feature workflow.

## How to invoke

**Claude Code:** `/skills/retrospective`

**Copilot (VS Code):** reference `.github/prompts/skills/retrospective.prompt.md`

## Related

- [Doc Producer](../doc-producer/) — previous stage
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
- [Addendum C: Feature Workflow](../../adr/platform/addendum-c-feature-workflow/) — full workflow diagram
