---
title: "Skill: Ticket Definer"
description: "Breaks an approved PRD and merged ADR into GitHub Issues ready for development."
summary: "Plays the role of Engineering Lead to create a complete, non-overlapping set of GitHub Issues with sizing, dependencies, and acceptance criteria."
tags: ["skill", "tickets", "planning", "github"]
date: 2026-05-15
---

## What this skill does

The Ticket Definer skill takes an approved PRD and a merged ADR and creates a complete set
of GitHub Issues in the service repository. It previews the full Issue set for engineer
review before creating anything, then creates all Issues via the GitHub MCP server.

The AI plays the role of an Engineering Lead. It checks for existing Issues first to avoid
duplication.

## When to use it

After the ADR has been merged to engineering-docs. Do not run this skill before the ADR is
merged — the architectural decisions must be settled before work is broken into tickets.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Approved PRD | Yes | The acceptance criteria to break into tickets |
| Merged ADR | Yes | The architecture to implement; ADR must be merged, not just drafted |
| Service repository name | Yes | The GitHub repo where Issues should be created |

## Outputs

- Preview of all Issues for engineer review (before any are created)
- GitHub Issues created in the service repository, each labelled `needs-review`
- Issues sized S/M/L with explicit dependencies between them

## Stage gate

After creating Issues, the skill stops and tells the engineer to review and label each
Issue as `ready-for-dev` before any implementation begins. No Copilot cloud agent
assignment should happen until the engineer has reviewed all Issues.

## How to invoke

**Claude Code:** `/skills/ticket-definer`

**Copilot (VS Code):** reference `.github/prompts/skills/ticket-definer.prompt.md`

**Canonical source:** `.claude/prompts/skills/ticket-definer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [System Designer](../system-designer/) — previous stage
- [Test Case Writer](../test-case-writer/) — next stage (after implementation)
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
- [Addendum C: Feature Workflow](../../adr/platform/addendum-c-feature-workflow/) — details on Copilot cloud agent assignment
