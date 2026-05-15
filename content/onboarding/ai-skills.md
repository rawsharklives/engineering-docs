---
title: "AI Skills: Getting Started"
description: "How to install and use the AI skills layer for structured, AI-assisted feature development."
summary: "Install AI skills into your service repo and learn how to use them to work through a complete feature without writing code."
date: 2026-05-15
---

## What are AI skills?

Skills are executable prompt definitions that give the AI a named role at each stage of
feature development. Instead of prompting the AI from scratch each time, you invoke a
named skill — `/skills/prd-writer`, `/skills/qa-reviewer`, etc. — and the AI knows
exactly what role it is playing, what it needs from you, what it will produce, and when to
stop and hand back to you.

The result is a structured, human-reviewed workflow where you review what the AI produces
at each stage before it moves on. You do not write code; you approve artefacts.

For the design decisions behind this, see [ADR-002: AI Skills Layer](../adr/platform/adr-002-ai-skills-layer/).

---

## Setup: Claude Code users

**Step 1 — one-time global setup** (once per machine):

```
/setup-skills-local
```

This installs `/setup-skills` into `~/.claude/commands/` so it's available in every repo.

**Step 2 — per-repo setup:**

Open Claude Code in the service repo and run:

```
/setup-skills
```

This installs:
- `.claude/commands/skills/` — Claude Code slash commands for each skill
- `.github/prompts/skills/` — Copilot prompt files for each skill

Commit both directories. The Copilot cloud agent needs the prompt files committed locally.

---

## Setup: Copilot-only users

No Claude Code required. In Copilot Chat, reference the setup prompt from
`engineering-docs` directly:

> "Using the GitHub MCP server, fetch `templates/copilot-prompts/setup-skills.prompt.md`
> from rawsharklives/engineering-docs (ref: main) and follow the instructions in it."

Copilot will fetch and write all 7 skill files to `.github/prompts/skills/` in the
current repo. Commit the result — done.

Re-run this whenever you want to pick up updates from `engineering-docs`.

---

## Running a skill

**Claude Code:**

```
/skills/prd-writer
```

The AI will ask for the inputs it needs (e.g. the feature brief) if you haven't provided
them.

**Copilot (VS Code):** reference the prompt file in Copilot Chat:

```
#.github/prompts/skills/prd-writer.prompt.md
```

Or use `@workspace` and reference the file name.

---

## The feature workflow at a glance

| Stage | Skill | What you provide | What you get back | Gate |
|-------|-------|-----------------|-------------------|------|
| 1 | `/skills/prd-writer` | Feature brief | PRD | Review in chat |
| 2 | `/skills/system-designer` | Approved PRD, service name | Draft ADR | Merge ADR PR |
| 3 | `/skills/ticket-definer` | PRD, merged ADR, repo name | GitHub Issues | Label ready-for-dev |
| 4 | Copilot cloud agent | Issues labelled ready-for-dev | Code PR(s) | PR review + CI |
| 5 | `/skills/test-case-writer` | PRD, code PR link | Test scenarios | Review in chat |
| 6 | `/skills/qa-reviewer` | PRD, code PR, test scenarios | Gap analysis | P0 gaps block merge |
| 7 | `/skills/doc-producer` | Merged ADR, code PR | Doc PRs | Merge doc PRs |
| 8 | `/skills/retrospective` | Brief, PRD, ADR, Issue data | Retro PR | Team reviews |

Full detail on every stage is in
[Addendum C: AI-Assisted Feature Workflow](../adr/platform/addendum-c-feature-workflow/).

---

## Assigning work to Copilot cloud agent (Stage 4)

After ticket-definer has created Issues and you have reviewed and labelled them
`ready-for-dev`:

1. Open the Issue in GitHub
2. In the **Assignees** field, assign the Issue to the Copilot cloud agent
3. Copilot reads the Issue, your `AGENTS.md`, and the merged ADR, then opens a draft PR
4. Review the PR — Copilot cannot merge its own PRs

Your service repo needs an `AGENTS.md` that tells Copilot where to find the service
documentation and ADR. See
[Addendum A: Copilot + GHEC Variant](../adr/platform/addendum-a-copilot-ghec-variant/)
for the `AGENTS.md` format.

---

## Keeping skills up to date

**Claude Code skills** update automatically — the thin-file pointer fetches the latest
canonical prompt from engineering-docs at each invocation. No action needed.

**Copilot prompt files** are committed locally and will go stale when engineering-docs
updates the canonical prompts. Re-run `/setup-skills` (Claude Code) or the Copilot-only
setup prompt and commit the updated files to pick up the latest versions.

---

## Browse all skills

Each skill has a reference page explaining its inputs, outputs, and stage gate:

- [PRD Writer](../skills/prd-writer/)
- [System Designer](../skills/system-designer/)
- [Ticket Definer](../skills/ticket-definer/)
- [Test Case Writer](../skills/test-case-writer/)
- [QA Reviewer](../skills/qa-reviewer/)
- [Doc Producer](../skills/doc-producer/)
- [Retrospective](../skills/retrospective/)
