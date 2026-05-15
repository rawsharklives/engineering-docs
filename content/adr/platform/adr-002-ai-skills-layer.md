---
title: "ADR-002: AI Skills Layer"
date: 2026-05-15
draft: false
---

# ADR-002: AI Skills Layer

| | |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-05-15 |
| **Author** | Rich |
| **Reviewers** | Platform |

---

## Context

ADR-001 established a passive knowledge base: documentation stored in Git, retrieved by AI
via MCP, and rendered for humans via Hugo. It defines how engineers and AI agents find and
consume knowledge — but it does not define how AI should behave during the software
development lifecycle itself.

In practice, teams are reaching for AI ad-hoc at each stage of feature development: writing
PRDs in one chat session, doing system design in another, breaking work into tickets in a
third. There is no shared prompt quality bar, no consistency across teams, no stage gate
that ensures one stage's output is ready before the next begins, and no single place to
update how the AI should behave when the process evolves.

The result is that AI assistance is available but not systematic. The knowledge base exists,
but the workflow does not.

This ADR introduces a **canonical AI Skills Layer** on top of ADR-001: a set of named,
versioned, machine-executable prompt definitions — called skills — for seven roles in the
software development lifecycle. Skills are not documentation pages. They are instructions
the AI runs, not information it retrieves.

### Skills vs doc pages

| | Doc pages (`content/**/*.md`) | Skills (`.claude/prompts/skills/*.md`) |
|---|---|---|
| **Nature** | Passive reference | Executable instructions |
| **AI relationship** | Retrieved as context | Run as a program |
| **Output** | Understanding | Artefacts |
| **Causal direction** | Skills reference doc pages | Doc pages do not reference skills |

The decision rule for authors: if the AI output is a deliverable artefact (PRD, GitHub
Issues, test scenarios), it belongs in a skill. If the output is understanding (what a
service does, what a naming convention is), it belongs in a doc page.

Skills may reference doc pages — they instruct the AI to retrieve specific pages via MCP
before producing output. Doc pages never reference skills; that causal direction couples
operational documentation to tooling choices that will change.

### Cross-repo without a mono-repo

ADR-001 established the thin-file-fetches-canonical pattern for `/docs` and `/work`: service
repos hold a one-line command file that fetches the real prompt from `engineering-docs` at
invocation time. This means canonical prompts can be updated centrally without touching
every service repo.

This ADR extends that pattern directly to skills. The canonical skill prompts live in
`.claude/prompts/skills/` in `engineering-docs`. Service repos hold thin one-line pointers
in `.claude/commands/skills/`. No mono-repo is required.

---

## Decision

We will introduce a canonical skills layer in `engineering-docs` consisting of seven
named skill definitions covering the full software development lifecycle. Each skill
defines: the role the AI plays, the inputs it requires, the process it follows, the
artefacts it produces, the quality bar it must meet, and the stage gate instruction that
tells the engineer what to review before proceeding.

### The seven skills

| Skill ID | Role the AI plays | Primary artefact |
|---|---|---|
| `prd-writer` | Senior Product Manager | PRD with user stories, acceptance criteria, out-of-scope, open questions |
| `system-designer` | Staff Engineer | Draft ADR + Mermaid architecture diagram |
| `ticket-definer` | Engineering Lead | GitHub Issues created via MCP, labelled `needs-review` |
| `test-case-writer` | QA Lead | Test scenario document (Given/When/Then) |
| `qa-reviewer` | QA Director | Gap analysis report with P0–P3 severity classification |
| `doc-producer` | Technical Writer | PRs to engineering-docs (runbook, service page updates) |
| `retrospective` | Engineering Lead | Retro document PR (blameless, action items with owners) |

### Directory layout

Skills live under `.claude/` in `engineering-docs` to maintain the convention that
everything under `.claude/` is AI tooling, not human-readable content:

```
engineering-docs/
├── .claude/
│   └── prompts/
│       └── skills/                  ← canonical skill prompts (7 files)
│           ├── prd-writer.md
│           ├── system-designer.md
│           ├── ticket-definer.md
│           ├── test-case-writer.md
│           ├── qa-reviewer.md
│           ├── doc-producer.md
│           └── retrospective.md
├── templates/
│   ├── skill-template.md            ← template for authoring new skills
│   └── claude-commands/
│       ├── setup-skills.md          ← bootstrap command for service repos
│       └── skills/                  ← thin-file pointers (7 files, Claude Code)
│   └── copilot-prompts/
│       └── skills/                  ← full-content Copilot prompt files (7 files)
└── content/
    └── skills/                      ← Hugo-rendered human-readable skill reference pages
```

Human-readable documentation for each skill is in `content/skills/` — these are reference
pages for engineers, not the executable prompts.

### Naming conventions

Skill IDs use kebab-case role descriptions:

| Location | Pattern |
|---|---|
| Canonical prompt (engineering-docs) | `.claude/prompts/skills/<skill-id>.md` |
| Claude Code thin-file (service repo) | `.claude/commands/skills/<skill-id>.md` |
| Copilot prompt file (service repo) | `.github/prompts/skills/<skill-id>.prompt.md` |
| Hugo reference page (engineering-docs) | `content/skills/<skill-id>.md` |

The skill ID is identical across all locations.

### Skill prompt structure

Every canonical skill prompt contains six mandatory sections:

1. **Role** — what role the AI is playing
2. **Inputs expected** — what the AI requires before starting; what to ask for if missing
3. **Process** — step-by-step instructions, including which doc pages to fetch via MCP
4. **Outputs** — the exact artefacts to produce and their format
5. **Quality bar** — self-check checklist the AI must complete before delivering
6. **Stage gate** — explicit instruction to stop and tell the engineer what to review and
   what skill to invoke next

### Thin-file pattern for Claude Code (service repos)

Each skill command file in a service repo is a one-line pointer, identical in structure to
the existing `/docs` and `/work` commands:

```
Using the GitHub MCP server, read the file `.claude/prompts/skills/<skill-id>.md` from
the rawsharklives/engineering-docs repository (ref: main) and follow the instructions
in it exactly.
```

Updating the canonical prompt in `engineering-docs` propagates to all service repos on
next invocation — no per-repo PRs required.

### Copilot prompt files (service repos)

For Copilot interactive use in VS Code or JetBrains, the thin-file pointer approach also
works. For Copilot's cloud agent (which executes autonomously against GitHub Issues), the
full prompt content must be committed locally — the cloud agent cannot perform an MCP fetch
as its bootstrap step. Service repos therefore commit full-content Copilot prompt files
(deployed from `templates/copilot-prompts/skills/` via `/setup-skills`).

This is a known asymmetry: Claude Code picks up canonical prompt updates automatically;
Copilot cloud agent users must re-run `/setup-skills` to pull updated prompt content.

### Bootstrap command

A new `/setup-skills` command (parallel to the existing `/setup-docs`) installs all skill
command files into a service repo. A global `/setup-skills-local` command installs
`/setup-skills` itself into `~/.claude/commands/` so it is available in all repos.

---

## Consequences

### Positive

- **Consistent AI-assisted SDLC across all teams.** Every team's PRD, system design, and
  QA review follows the same quality bar, defined once in one place.
- **Centrally updatable.** Improving a skill prompt in `engineering-docs` propagates to all
  service repos on next invocation (Claude Code) or next `/setup-skills` run (Copilot).
- **No mono-repo required.** The thin-file pattern works across any number of service repos
  with no per-repo maintenance.
- **Works for both Claude Code and Copilot.** Same canonical prompts, two deployment paths.
- **Stage gates enforced by design.** Each skill stops and explicitly names the next step.
  Engineers cannot skip stages accidentally; skipping requires deliberate intent.
- **Artefact trail.** Each stage produces a concrete artefact. The full feature history
  (PRD → ADR → Issues → tests → QA → docs → retro) is traceable in GitHub.

### Negative

- **Engineers must run `/setup-skills` per service repo.** This is a one-time setup cost
  per repo, but it is a manual step that can be forgotten.
- **Copilot cloud agent requires committed full-content files.** These go stale when
  canonical prompts are updated. Teams using the cloud agent must re-run `/setup-skills`
  periodically.
- **Skills require maintenance.** As engineering processes evolve, skill prompts must be
  updated. The Platform team owns this; it is ongoing work, not a one-time cost.
- **Stage gate is process, not automation.** There is no CI enforcement between stages.
  An engineer could invoke `/system-designer` without an approved PRD. The gate relies on
  team discipline and is reinforced by the skill prompt text, not tooling.

### Neutral / follow-on work

- **Addendum C** documents the complete feature workflow: stage-by-stage inputs, outputs,
  gate mechanisms, and Copilot cloud agent integration. See
  [Addendum C: AI-Assisted Feature Workflow](../addendum-c-feature-workflow/).
- **`AGENTS.md` template** (Addendum A follow-on): when written, it should mention
  `/setup-skills` and describe available skills for the Copilot cloud agent.
- **Standards pages** for PRD format, story-writing conventions, and QA severity
  definitions should be added to `content/standards/` so skill prompts can reference
  them via MCP rather than embedding the content in the prompt.
- **Custom skill variants.** Teams may want role-specific variants (e.g. a `prd-writer`
  tuned for data platform features). The `templates/skill-template.md` supports this.
  Custom variants should be stored in the team's area of `engineering-docs` and referenced
  by a service-specific thin-file.

---

## Addendum: Further reading

| Addendum | Title | Status | Date |
|----------|-------|--------|------|
| [Feature workflow](../addendum-c-feature-workflow/) | AI-Assisted Feature Workflow | Proposed | 2026-05-15 |
