---
title: "AI Skills"
description: "Canonical AI role definitions for the software development lifecycle."
summary: "Named, versioned prompt definitions that give the AI a structured role at each stage of feature development."
---

Skills are executable prompt definitions that give the AI a named role in a structured
software development lifecycle. Each skill defines: the role the AI plays, the inputs it
expects, the artefacts it produces, the quality bar it must meet, and the stage gate that
tells the engineer what to review before the next stage begins.

Skills are invoked with `/skills/<name>` in Claude Code or by referencing the
`.github/prompts/skills/<name>.prompt.md` file in Copilot Chat.

**Skills are not documentation.** They are instructions the AI runs, not reference material
it retrieves. For an explanation of the distinction, see
[ADR-002: AI Skills Layer](../adr/platform/adr-002-ai-skills-layer/).

## The feature workflow

The seven skills map to the full feature development lifecycle:

| # | Skill | Role | Artefact |
|---|-------|------|----------|
| 1 | [PRD Writer](prd-writer/) | Senior PM | Product Requirements Document |
| 2 | [System Designer](system-designer/) | Staff Engineer | Draft ADR + architecture diagram |
| 3 | [Ticket Definer](ticket-definer/) | Engineering Lead | GitHub Issues |
| 4 | *(Copilot cloud agent writes code)* | — | Code PR |
| 5 | [Test Case Writer](test-case-writer/) | QA Lead | Test scenario document |
| 6 | [QA Reviewer](qa-reviewer/) | QA Director | Gap analysis report |
| 7 | [Doc Producer](doc-producer/) | Technical Writer | Runbook / service page PRs |
| 8 | [Retrospective](retrospective/) | Engineering Lead | Retro document PR |

For the complete workflow with stage gates and gate mechanisms, see
[Addendum C: AI-Assisted Feature Workflow](../adr/platform/addendum-c-feature-workflow/).

## Getting started

**Claude Code:** run `/setup-skills-local` once to install the `/setup-skills` command
globally, then run `/setup-skills` in any service repo to install all skill commands.
Commit the resulting files.

**Copilot only:** in Copilot Chat, ask it to fetch and run
`templates/copilot-prompts/setup-skills.prompt.md` from `rawsharklives/engineering-docs`.
It will install the 7 Copilot skill files into `.github/prompts/skills/`. Commit the
result.

Full setup instructions are in the [onboarding guide](../onboarding/ai-skills/).
