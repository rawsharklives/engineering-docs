---
title: "Addendum C: AI-Assisted Feature Workflow"
date: 2026-05-15
draft: false
---

# Addendum C: AI-Assisted Feature Workflow

| | |
|---|---|
| **Status** | Proposed |
| **Date** | 2026-05-15 |
| **Relates to** | [ADR-001](../adr-001-engineering-documentation-platform/), [ADR-002](../adr-002-ai-skills-layer/) |

---

## Purpose

This addendum documents the complete AI-assisted feature workflow built on the skills layer
introduced in ADR-002. It describes stage-by-stage inputs, outputs, gate mechanisms, and
the role of the Copilot cloud agent in autonomous implementation.

The workflow is a process, not a pipeline. There is no CI enforcement between stages. The
engineer is the orchestrator — each stage gate is the engineer's intentional decision to
invoke the next skill.

---

## Workflow overview

```
Feature Brief
     │
     ▼  /skills/prd-writer
┌──────────────────────────────────┐
│ Stage 1: Product Requirements    │
│ Artefact: PRD                    │
│ Gate: engineer approves in chat  │
└──────────┬───────────────────────┘
           │ approved
           ▼  /skills/system-designer
┌──────────────────────────────────┐
│ Stage 2: System Design           │
│ Artefact: Draft ADR + diagram    │
│ Gate: PR to engineering-docs →   │
│       CODEOWNERS review          │
└──────────┬───────────────────────┘
           │ ADR merged
           ▼  /skills/ticket-definer
┌──────────────────────────────────┐
│ Stage 3: Ticket Definition       │
│ Artefact: GitHub Issues          │
│ Gate: engineer labels            │
│       ready-for-dev              │
└──────────┬───────────────────────┘
           │ Issues approved
           ▼  assign to Copilot cloud agent
┌──────────────────────────────────┐
│ Stage 4: Implementation          │
│ Artefact: Code PR(s)             │
│ Gate: engineer PR review + CI    │
└──────────┬───────────────────────┘
           │ code PR approved
           ▼  /skills/test-case-writer
┌──────────────────────────────────┐
│ Stage 5: Test Cases              │
│ Artefact: Test scenario doc      │
│ Gate: engineer approves in chat  │
└──────────┬───────────────────────┘
           │ test cases approved
           ▼  /skills/qa-reviewer
┌──────────────────────────────────┐
│ Stage 6: QA Review               │
│ Artefact: Gap analysis report    │
│ Gate: P0 gaps block merge;       │
│       P1/P2 become Issues        │
└──────────┬───────────────────────┘
           │ QA approved (no P0 gaps)
           ▼  merge
           │
           ▼  /skills/doc-producer
┌──────────────────────────────────┐
│ Stage 7: Documentation           │
│ Artefact: PRs to engineering-docs│
│ Gate: CODEOWNERS review          │
└──────────┬───────────────────────┘
           │ doc PRs merged
           ▼  /skills/retrospective
┌──────────────────────────────────┐
│ Stage 8: Retrospective           │
│ Artefact: Retro doc PR           │
│ Gate: team reviews async;        │
│       action items → Issues      │
└──────────────────────────────────┘
```

---

## Stage-by-stage reference

### Stage 1 — Product Requirements

**Skill:** `prd-writer`
**Engineer provides:** feature brief (text, bullets, or voice transcript)
**AI produces:** PRD with user stories, acceptance criteria, out-of-scope, open questions,
success metrics
**Gate mechanism:** conversational — the skill stops and explicitly asks the engineer to
review. The engineer approves by saying "looks good" or requests revisions. No next skill
runs until the engineer explicitly invokes `/skills/system-designer`.
**Where artefact lives:** in the chat session; engineer pastes or links it when invoking
the next skill.

---

### Stage 2 — System Design

**Skill:** `system-designer`
**Engineer provides:** approved PRD; service name
**AI produces:** draft ADR in engineering-docs template format, including a Mermaid
architecture diagram; list of follow-on architectural questions
**Gate mechanism:** PR review — the skill instructs the engineer to raise the draft ADR as
a PR to `content/adr/platform/` in engineering-docs. CODEOWNERS enforces review by
Platform and relevant tech leads. The ADR must be merged before Stage 3 begins.
**Where artefact lives:** `content/adr/platform/adr-<NNN>-<slug>.md` in engineering-docs.

---

### Stage 3 — Ticket Definition

**Skill:** `ticket-definer`
**Engineer provides:** approved PRD; merged ADR link; service repository name
**AI produces:** preview of all Issues (title, description, acceptance criteria, size S/M/L,
dependencies); then creates Issues in the service repo via GitHub MCP, labelled
`needs-review`
**Gate mechanism:** two-step. First, the engineer reviews the Issue preview and confirms
before any Issues are created. Second, after Issues are created, the engineer reviews them
in GitHub and changes the label from `needs-review` to `ready-for-dev` for each approved
Issue. No implementation begins until `ready-for-dev` is applied.
**Where artefact lives:** GitHub Issues in the service repository.

---

### Stage 4 — Implementation (Copilot cloud agent)

**No skill** — this stage is executed by the Copilot cloud agent or by Claude Code in
work mode, working against the `ready-for-dev` Issues.

**How the Copilot cloud agent is invoked:**
1. In GitHub, open each Issue labelled `ready-for-dev`
2. In the Assignees field, assign the Issue to the Copilot cloud agent
3. Copilot reads the Issue description, reads `AGENTS.md` in the service repo (which
   points to relevant ADRs and service docs in engineering-docs), and opens a draft PR

**What the cloud agent reads:**
- The GitHub Issue (title, description, acceptance criteria)
- `AGENTS.md` in the service repo — this file should reference the merged ADR and the
  service page in engineering-docs
- Files in the service repo via the built-in GitHub MCP server

**Gate mechanism:** standard PR review — the engineer reviews the draft PR, CI must pass,
and CODEOWNERS must approve before merge. The cloud agent cannot merge its own PRs.

**Where artefact lives:** draft PR in the service repository.

---

### Stage 5 — Test Cases

**Skill:** `test-case-writer`
**Engineer provides:** approved PRD; code PR link (the merged or open PR from Stage 4)
**AI produces:** test scenario document with per-criterion Given/When/Then scenarios
(happy path, edge case, error) and cross-cutting scenarios (security, performance)
**Gate mechanism:** conversational — the skill stops and asks the engineer to review for
coverage gaps before running `qa-reviewer`.
**Where artefact lives:** in the chat session; engineer provides it when invoking the next
skill.

---

### Stage 6 — QA Review

**Skill:** `qa-reviewer`
**Engineer provides:** approved PRD; code PR link; test scenario document
**AI produces:** gap analysis report with per-criterion findings classified P0–P3; verdict
of Approve / Conditional Approve / Block; cross-cutting findings on auth, error handling,
and observability
**Gate mechanism:**
- **Approve or Conditional Approve (no P0):** engineer may proceed to merge. P1/P2 items
  must be logged as Issues before release.
- **Block (P0 present):** merge is blocked. Engineer creates a GitHub Issue for each P0
  gap, links it to the PR, and re-runs `/skills/qa-reviewer` after resolution.
**Where artefact lives:** in the chat session; may be posted as a PR comment by the
engineer for the review record.

---

### Stage 7 — Documentation

**Skill:** `doc-producer`
**Engineer provides:** merged ADR; merged code PR summary; service name
**AI produces:** PRs to engineering-docs containing updated service page, new/updated
runbooks, onboarding updates
**Gate mechanism:** PR review — CODEOWNERS enforces review before merge. The skill raises
PRs; it does not push to main. Separate PRs for logically separate changes (e.g. service
page update vs new runbook).
**Where artefact lives:** PRs to `content/services/`, `content/runbooks/`, and
`content/onboarding/` in engineering-docs.

---

### Stage 8 — Retrospective

**Skill:** `retrospective`
**Engineer provides:** original feature brief; merged PRD; merged ADR; Issue delivery data
(which Issues closed, on/over/under estimate); optionally QA gap analysis
**AI produces:** blameless retro document (planned vs shipped, what went well/poorly,
action items with owners, lessons learned) as a PR to `content/rca/` in engineering-docs
**Gate mechanism:** async team review. Engineers review the PR, agree on action items, and
create GitHub Issues for each before closing. This is the final stage gate.
**Where artefact lives:** `content/rca/<YYYY-MM-DD>-<feature-slug>.md` in engineering-docs.

---

## Gate mechanism summary

| Stage | Gate type | Who enforces |
|-------|-----------|--------------|
| 1 — PRD | Conversational approval | Engineer |
| 2 — System Design | CODEOWNERS PR review | Platform / tech leads |
| 3 — Tickets | GitHub Issue label change | Engineer |
| 4 — Implementation | PR review + CI | Engineer + CODEOWNERS |
| 5 — Test Cases | Conversational approval | Engineer |
| 6 — QA Review | P0 verdict blocks merge | Engineer |
| 7 — Documentation | CODEOWNERS PR review | Service owner / Platform |
| 8 — Retrospective | Team async PR review | Team |

---

## Artefact registry

| Stage | Artefact | Location |
|-------|----------|----------|
| 1 | PRD | Chat session |
| 2 | ADR | `content/adr/platform/` in engineering-docs |
| 3 | GitHub Issues | Service repository Issues |
| 4 | Code PR | Service repository PRs |
| 5 | Test scenario doc | Chat session |
| 6 | Gap analysis report | Chat session (optionally PR comment) |
| 7 | Service page, runbooks | `content/` in engineering-docs |
| 8 | Retro document | `content/rca/` in engineering-docs |

---

## Using Claude Code vs Copilot

Both tools can run all eight skills. The choice of tool per stage is a matter of
preference, not architecture. A team may use Claude Code for Stages 1–3 and 5–8 (which
are interactive reasoning tasks) and Copilot cloud agent for Stage 4 (which is autonomous
file-writing from a specification).

**Why the cloud agent is suited to Stage 4:**
- Stage 4 has a complete specification: the GitHub Issue with acceptance criteria and the
  merged ADR
- The cloud agent can work asynchronously — the engineer does not need to be present
- The output (a code PR) is reviewed by the engineer before it has any effect
- The cloud agent cannot merge its own PR — the human gate is structurally enforced

**Why the cloud agent is less suited to Stages 1–3 and 5–8:**
- These stages require iterative reasoning about ambiguous inputs (a feature brief, a QA
  finding, a retrospective observation)
- They benefit from the engineer being present to redirect or refine
- The interactive chat model (Claude Code or Copilot Chat) is better suited

---

## Consequences

### Positive

- Every feature has a complete, traceable artefact trail: brief → PRD → ADR → Issues →
  code → tests → QA → docs → retro
- Engineers review and approve at every stage; no stage can be skipped without deliberate
  intent
- The Copilot cloud agent is scoped to the one stage where autonomous execution is safe:
  implementation against a settled specification with a human PR review gate

### Negative

- The workflow is sequential; some teams may want to run stages in parallel (e.g. test
  case writing during implementation). This is permitted — the stage gate is a process
  recommendation, not a technical block.
- Small features (one-line bug fixes, config changes) do not benefit from the full
  workflow. Teams should use judgement and skip stages that add no value for the scope.
- The chat-session artefacts (PRD, test scenarios, QA report) are not automatically
  persisted. Engineers must save them or post them as PR comments to maintain the
  artefact trail.

### Neutral / follow-on

- A lightweight wrapper (GitHub Action or Copilot Extension) could automate the artefact
  persistence problem: capturing skill outputs as PR comments or engineering-docs PRs
  automatically. This is deferred until the manual workflow has proven its value.
- `AGENTS.md` template for service repos (Addendum A follow-on) should reference the
  feature workflow and list available skills for the cloud agent's context.
