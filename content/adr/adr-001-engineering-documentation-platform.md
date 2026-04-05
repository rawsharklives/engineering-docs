---
title: "ADR-001: Engineering Documentation Platform"
date: 2026-04-01
draft: false
tags: ["platform", "documentation", "hugo", "mcp"]
---

# ADR-001: Engineering Documentation Platform

**Status:** Proposed
**Date:** April 2026
**Author:** Rich — VP Engineering
**Reviewers:** Platform Engineering Team

---

## Context

Our current engineering knowledge is fragmented and unreliable. Documentation is spread
across the DevOps Wiki, individual Confluence pages, Notion, and informal Slack threads —
with no single source of truth. Content is frequently stale, inconsistently formatted, and
hard to discover. Engineers waste time searching multiple systems for runbooks, architecture
decisions, and onboarding guides, and cannot trust that what they find is current.

There is no enforced structure around critical operational knowledge. Runbooks exist in
different formats. Architecture decisions are rarely written down, and when they are, they
live in the heads of individuals rather than a shared, searchable record. New joiners in
particular struggle, as onboarding material is scattered and out of date.

The current state also means Claude Code — and AI tooling more broadly — has no reliable
knowledge base to reference. Without a canonical, machine-readable docs store, AI-assisted
workflows have nothing authoritative to draw from.

## Decision

We will establish a docs-as-code platform built on four integrated components:

**1. Canonical Docs Repository (GHEC)**
A dedicated GitHub repository (`netwealth/engineering-docs`) in GitHub Enterprise Cloud
becomes the single source of truth for all engineering knowledge. All documentation is
written in Markdown and lives under version control. The repository is structured around
four document types: runbooks, architecture decision records (ADRs), onboarding guides,
and postmortems. Templates are provided for each type and enforced via CODEOWNERS and
PR review.

**2. Static Site Wiki (Hugo + GitHub Pages)**
A Hugo-based static site renders the Markdown repository as a browsable internal wiki,
deployed automatically to GitHub Pages via GitHub Actions on every merge to `main`. This
gives the team a clean, searchable, human-readable interface to the docs without requiring
Git knowledge to read them.

**3. CLAUDE.md Instruction Layer**
Each engineer's Claude Code environment is configured with a `CLAUDE.md` file that
establishes context about the docs platform — where docs live, what document types exist,
which templates to use, and how to interact with the canonical store. A repo-level
`CLAUDE.md` is committed to `engineering-docs` and shared across the team. Engineers may
also maintain a local, gitignored override for personal preferences.

**4. MCP Integration (GitHub MCP Server)**
Anthropic's GitHub MCP server is configured to give Claude Code programmatic access to
`engineering-docs`. This enables Claude to search, read, and propose changes to
documentation directly from within any engineering workflow — finding existing runbooks
before generating new ones, reading ADRs for architectural context, and raising PRs when
new documentation is needed.

**Repository structure:**

```
engineering-docs/
├── CLAUDE.md
├── content/
│   ├── runbooks/
│   ├── adr/
│   ├── onboarding/
│   ├── postmortems/
│   └── standards/
├── templates/
│   ├── runbook-template.md
│   ├── adr-template.md
│   └── postmortem-template.md
├── hugo.toml
└── .github/
    └── workflows/
        └── deploy.yml
```

## Consequences

### Positive

- **Single source of truth.** All engineering documentation lives in one place, under
  version control, with a clear structure and enforced templates.
- **Documentation as a first-class engineering practice.** Docs go through the same
  review process as software — PRs, CODEOWNERS approval, and a clear audit trail.
- **Improved discoverability.** The Hugo wiki gives the whole team a fast, searchable,
  always-current interface to operational knowledge.
- **AI-ready knowledge base.** The MCP integration means Claude Code can reference
  canonical documentation during development workflows.
- **Low operational overhead.** No new infrastructure. GitHub Enterprise Cloud, GitHub
  Actions, and GitHub Pages are tools we already have.
- **Portability and longevity.** Plain Markdown in Git is readable by any tool, any
  editor, and any future platform.

### Negative

- **Adoption requires discipline.** The value is entirely dependent on consistent use.
  Without team buy-in, the repo risks becoming another stale documentation graveyard.
- **Git as the contribution interface.** Non-engineering stakeholders will need guidance
  or a lightweight editor workflow.
- **Initial migration effort.** Existing useful content in the DevOps Wiki and other
  systems needs to be reviewed, updated, and migrated.
- **MCP is relatively new tooling.** The GitHub MCP server integration requires
  familiarity and per-engineer configuration overhead.
- **Hugo theme maintenance.** Significant customisation would become an asset to maintain.
  Keeping it minimal reduces this risk.

### Follow-on work

- Configure GitHub MCP server for each engineer's Claude Code environment
- Migrate high-value content from DevOps Wiki and Confluence
- Define a process for keeping docs current (ownership, review cadence)
- Evaluate onboarding editor tooling for non-Git contributors
