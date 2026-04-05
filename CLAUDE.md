# Engineering Docs — Claude Code Context

This repository is the canonical source of truth for engineering knowledge at [company-name].
All documentation is written in Markdown, version-controlled in Git, and rendered as a
static wiki via Hugo + GitHub Pages.

## What lives here

| Section | Path | Purpose |
|---------|------|---------|
| Runbooks | `content/runbooks/` | Step-by-step operational procedures |
| ADRs | `content/adr/` | Architecture Decision Records |
| Onboarding | `content/onboarding/` | Guides for new engineers |
| RCAs | `content/rca/` | Root Cause Analyses |
| Standards | `content/standards/` | Engineering standards and conventions |

Templates for each document type are in `templates/`.

## How to work with this repo

**Before creating new documentation:**
Search this repository for existing content first. Use the GitHub MCP server or grep to
check whether a runbook, ADR, or guide already exists before writing a new one.

**Finding docs:**
- Runbooks: `content/runbooks/`
- ADRs: `content/adr/` — numbered sequentially, e.g. `adr-001-*.md`
- Onboarding: `content/onboarding/`
- RCAs: `content/rca/`

**Creating new documents:**
Copy the appropriate template from `templates/` and fill it in. Follow the naming
conventions below.

## Naming conventions

- Runbooks: `content/runbooks/<service>-<action>.md` (e.g. `payments-rollback.md`)
- ADRs: `content/adr/adr-<NNN>-<slug>.md` (e.g. `adr-002-api-gateway.md`)
- RCAs: `content/rca/<YYYY-MM-DD>-<slug>.md`
- Onboarding: `content/onboarding/<topic>.md`

## Proposing changes

All changes go through a pull request. CODEOWNERS requires review before merge.
When a user asks you to create or update documentation, raise a PR rather than pushing
directly to `main`.

## Hugo site

The rendered wiki is at: https://rawsharklives.github.io/engineering-docs/

Hugo config: `hugo.toml`
Theme: PaperMod (submodule at `themes/PaperMod`)
Build and deploy: `.github/workflows/deploy.yml` (triggers on push to `main`)
