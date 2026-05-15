---
title: "Skill: QA Reviewer"
description: "Cross-references a feature's PRD, implementation, and test scenarios to produce a gap analysis with a clear Approve/Block verdict."
summary: "Plays the role of QA Director to identify gaps and contradictions between specification, implementation, and test coverage."
tags: ["skill", "qa", "review"]
date: 2026-05-15
---

## What this skill does

The QA Reviewer skill takes the PRD, the code PR, and the test scenario document, and
cross-references them to find: acceptance criteria not implemented, criteria not tested,
and contradictions between specification and implementation.

The AI plays the role of a QA Director. It does not run tests — it reviews artefacts.

Every gap is classified P0 (blocking) through P3 (nice-to-have). The verdict is:
- **Approve** — P2/P3 gaps only; no blocking or must-fix issues
- **Conditional Approve** — P1 gaps present; must be logged as Issues before release
- **Block** — P0 gaps present; must be resolved before merge

## When to use it

After test cases have been written and reviewed. The test scenario document is a required
input — QA review without it cannot verify test coverage.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Approved PRD | Yes | The baseline acceptance criteria |
| Code PR link | Yes | The implementation to review |
| Test scenario document | Yes | Output of the Test Case Writer skill |
| Merged ADR | No | Useful for checking architectural constraints |

## Outputs

A gap analysis report containing:
- Verdict: Approve / Conditional Approve / Block
- Per-criterion gap analysis table with P0–P3 severity
- Cross-cutting findings on auth/authz, error handling, and observability
- Specific action required for P0 gaps

## Stage gate

If verdict is Approve or Conditional Approve: the skill tells the engineer to merge and
then run [Doc Producer](../doc-producer/).

If verdict is Block: the skill tells the engineer to create Issues for P0 gaps and re-run
this skill after they are resolved.

## How to invoke

**Claude Code:** `/skills/qa-reviewer`

**Copilot (VS Code):** reference `.github/prompts/skills/qa-reviewer.prompt.md`

**Canonical source:** `.claude/prompts/skills/qa-reviewer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [Test Case Writer](../test-case-writer/) — previous stage
- [Doc Producer](../doc-producer/) — next stage (after merge)
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
