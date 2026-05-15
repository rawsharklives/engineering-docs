---
title: "Skill: Test Case Writer"
description: "Writes acceptance test scenarios for a completed implementation."
summary: "Plays the role of QA Lead to produce a structured test scenario document covering happy path, edge cases, errors, security, and performance."
tags: ["skill", "qa", "testing"]
date: 2026-05-15
---

## What this skill does

The Test Case Writer skill reads a completed code PR and the original PRD, then produces a
comprehensive set of acceptance test scenarios. It reads the service page from
engineering-docs for SLO context when writing performance scenarios.

The AI plays the role of a QA Lead. It produces test scenarios (Given/When/Then format),
not test code. Developers or QA engineers execute the scenarios.

## When to use it

After implementation is complete and a code PR exists. The PRD and a code PR link are both
required — the skill cross-references what was specified against what was implemented.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Approved PRD | Yes | The acceptance criteria to generate test scenarios from |
| Code PR link | Yes | The PR or branch to review for implemented behaviour |
| Merged ADR | No | Helps identify edge cases specific to the architectural approach |
| Service name | No | Used to read SLO targets for performance scenarios |

## Outputs

A test scenario document containing:
- Per-criterion scenarios in Given/When/Then format (happy path, edge case, error)
- Cross-cutting scenarios for security, performance, and concurrent access
- Scenario IDs (TC-N-M format) for traceability

## Stage gate

The skill stops after producing the document and tells the engineer to review for coverage
gaps before running [QA Reviewer](../qa-reviewer/).

## How to invoke

**Claude Code:** `/skills/test-case-writer`

**Copilot (VS Code):** reference `.github/prompts/skills/test-case-writer.prompt.md`

**Canonical source:** `.claude/prompts/skills/test-case-writer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [Ticket Definer](../ticket-definer/) — previous stage
- [QA Reviewer](../qa-reviewer/) — next stage
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
