---
title: "Skill: Doc Producer"
description: "Produces post-ship documentation for a completed feature: updated service page, new runbooks, and onboarding updates."
summary: "Plays the role of Technical Writer to update engineering-docs after a feature ships, raising PRs for service page, runbook, and onboarding changes."
tags: ["skill", "documentation", "runbooks"]
date: 2026-05-15
---

## What this skill does

The Doc Producer skill reads the merged ADR, the code PR, and the current service page in
engineering-docs, then produces all documentation updates needed to reflect the new state
of the service. It raises PRs to engineering-docs — it does not push directly to main.

The AI plays the role of a Technical Writer. All changes go through CODEOWNERS review.

## When to use it

After the feature has merged to the service repository. The ADR must already be merged to
engineering-docs. Run this skill before closing the feature milestone.

## Inputs

| Input | Required | Description |
|-------|----------|-------------|
| Merged ADR | Yes | Architectural context; must be merged to engineering-docs |
| Merged code PR | Yes | What changed in the service |
| Service name | Yes | Locates the existing service page in engineering-docs |
| New operational procedures | No | Any new deploy, rollback, or alert-response steps |

## Outputs

PRs to engineering-docs for:
- Updated `content/services/<service>/_index.md` (new capabilities, changed interfaces)
- New runbooks in `content/runbooks/<service>-<action>.md` (for new operational procedures)
- Updated runbooks (for changed operational procedures)
- Updated onboarding guides (if the feature changes the new-engineer experience)

## Stage gate

After raising PRs, the skill stops and tells the engineer to merge them via the standard
CODEOWNERS review process, then run [Retrospective](../retrospective/).

## How to invoke

**Claude Code:** `/skills/doc-producer`

**Copilot (VS Code):** reference `.github/prompts/skills/doc-producer.prompt.md`

**Canonical source:** `.claude/prompts/skills/doc-producer/SKILL.md` in rawsharklives/engineering-docs (agentskills.io format)

## Related

- [QA Reviewer](../qa-reviewer/) — previous stage
- [Retrospective](../retrospective/) — next stage
- [Runbook template](../../runbooks/) — the format new runbooks must follow
- [ADR-002: AI Skills Layer](../../adr/platform/adr-002-ai-skills-layer/)
