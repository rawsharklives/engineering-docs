---
title: "Skill: System Designer"
description: "Produces a draft ADR and architecture design from an approved PRD."
summary: "Plays the role of Staff Engineer to produce a draft ADR with Mermaid diagram and list of follow-on architectural decisions."
tags: ["skill", "architecture", "adr", "design"]
date: 2026-05-15
---

## What this skill does

The System Designer skill takes an approved PRD and produces a draft Architecture Decision
Record (ADR) ready for tech lead review. It reads the current service page and existing
ADRs from engineering-docs via MCP before designing, ensuring the approach does not
contradict existing decisions.

The AI plays the role of a Staff Engineer. It does not write code — it decides the
architectural approach and documents the reasoning.

## When to use it

After the PRD has been reviewed and approved. The PRD must exist before this skill runs —
system design without a clear specification produces unreviewable output.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Approved PRD | Yes | Confirmed as reviewed by the engineer |
| Service name | Yes | So the skill can read the current service page from engineering-docs |
| Constraints or existing ADRs | No | Architectural decisions that must not be contradicted |

## Outputs

- Complete draft ADR in the engineering-docs ADR template format
- At least one Mermaid architecture diagram embedded in the ADR
- Bullet list of follow-on ADRs or unresolved architectural questions

## Stage gate

The skill stops after producing the draft ADR and tells the engineer to raise it as a PR
to `content/adr/platform/` in engineering-docs for CODEOWNERS review. It will not proceed
to ticket definition — the ADR must be merged first.

## How to invoke

**Claude Code:** `/skills/system-designer`

**Copilot (VS Code):** reference `.github/prompts/skills/system-designer.prompt.md`

**Canonical source:** `.claude/prompts/skills/system-designer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [PRD Writer](../prd-writer/) — previous stage
- [Ticket Definer](../ticket-definer/) — next stage (after ADR is merged)
- [ADR template](../../templates/) — the format the skill follows
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
