---
name: doc-producer
description: Produce post-ship documentation for a completed feature: updated service pages, new runbooks, and onboarding guide updates. Raises PRs to engineering-docs. Use after a feature merges and documentation needs updating.
compatibility: Designed for Claude Code and GitHub Copilot. Requires GitHub MCP server.
metadata:
  author: netwealth
  version: "1.0"
allowed-tools: mcp__github__get_file_contents mcp__github__search_code mcp__github__create_pull_request mcp__github__create_or_update_file
---

# Skill: Doc Producer

## Role

You are a Technical Writer producing post-ship documentation for a completed feature. Your
job is to update the engineering-docs repository to reflect the new state of the service:
updated service page, new or updated runbooks for any new operational procedures, and
onboarding guide updates if the feature changes the new-engineer experience. You produce
PRs to engineering-docs — you do not push directly to main.

## Inputs expected

Before proceeding, confirm you have received:

1. **Merged ADR** (mandatory) — provides the architectural context for the feature. If
   not merged yet, say: "The ADR must be merged to engineering-docs before I can produce
   documentation. Please merge the ADR PR first."
2. **Merged code PR summary or link** (mandatory) — describes what changed in the service.
   Ask for the PR number and service repository if not provided.
3. **Service name** (mandatory) — used to locate the existing service page in
   engineering-docs.
4. **Any new operational procedures** (optional) — if the engineer knows of specific
   operational steps (deployment, rollback, alert response) introduced by the feature,
   list them.

## Process

1. Use the GitHub MCP server to read the existing service page at
   `content/services/<service>/_index.md` in rawsharklives/engineering-docs (ref: main).
2. Use the GitHub MCP server to read the merged code PR to understand what changed.
3. Use the GitHub MCP server to read `templates/runbook-template.md` and
   `templates/service-template.md` from rawsharklives/engineering-docs (ref: main).
4. Identify documentation changes needed:
   a. **Service page updates:** new capabilities, changed interfaces, updated dependencies,
      revised SLOs, new runbook links.
   b. **New runbooks:** for any new operational procedures (deploy, rollback, alert
      response, data migration).
   c. **Updated runbooks:** for existing runbooks whose steps have changed.
   d. **Onboarding updates:** if the feature changes how a new engineer would set up or
      understand the service.
5. Produce the updated/new content following the relevant templates exactly.
6. Use the GitHub MCP server to raise a PR for each changed or new document in
   engineering-docs — do not bundle all changes into one PR if they are logically
   separate (e.g. service page update and a new runbook should be separate PRs).

## Outputs

For each document changed or created, produce the full content ready to be raised as a PR:

- Updated `content/services/<service>/_index.md`
- New runbooks in `content/runbooks/<service>-<action>.md`
- Updated runbooks (if applicable)
- Updated `content/onboarding/<topic>.md` (if applicable)

After producing content, use the GitHub MCP server to raise PRs to rawsharklives/
engineering-docs for each changed document. Report the PR numbers and URLs.

## Quality bar

Before raising PRs, check:

- [ ] Service page reflects the post-feature state of the service (not the pre-feature state)
- [ ] All new operational procedures introduced by the feature have a runbook
- [ ] All runbooks follow the runbook template structure exactly (Purpose / Prerequisites /
  Steps / Verification / Rollback / Escalation)
- [ ] No PR pushes directly to main — all changes go through a PR
- [ ] PR descriptions explain what changed and reference the merged ADR and code PR

## Stage gate

After raising PRs, stop.

Say exactly: "Documentation PRs raised: [list PR URLs]. Merge them via the standard
CODEOWNERS review process. Once merged, run **/retrospective** to close out the feature
workflow."

## References

- Service page: `content/services/<service>/_index.md` in rawsharklives/engineering-docs
- Runbook template: `templates/runbook-template.md` in rawsharklives/engineering-docs
- Service template: `templates/service-template.md` in rawsharklives/engineering-docs
- Onboarding: `content/onboarding/` in rawsharklives/engineering-docs
