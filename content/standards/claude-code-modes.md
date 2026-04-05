---
title: "Claude Code Modes: /docs and /work"
description: "How to use and set up the /docs and /work Claude Code slash commands across service repos."
---

Claude Code supports two modes to help engineers switch between querying engineering
documentation and working on a service repo. These modes are controlled by slash commands
(`/docs` and `/work`) and are designed to be set up once per service repo, with all
prompt behaviour managed centrally from this repository.

## The two modes

### /docs — Documentation mode

Instructs Claude to answer exclusively from this repository (`engineering-docs`) via the
GitHub MCP server. Claude will not use local repo files, training knowledge, or the web.
If the answer is not in engineering-docs, it will say so rather than guessing.

Use this when:
- Looking up a runbook, ADR, or service page
- Asking questions about platform architecture or engineering standards
- Verifying what the canonical documentation says before making a decision

### /work — Work mode

Returns Claude to normal engineering mode. Local repo files, training knowledge, and all
available tools are in scope. The engineering-docs repo remains available via MCP for
reference when needed.

Use this when:
- Writing or debugging code
- Working on tasks within the service repo
- Using Claude for general engineering assistance

## Switching between modes

| Command | Effect |
|---------|--------|
| `/docs` | Switch to Documentation mode |
| `/work` | Switch to Work mode |
| `/clear` | Clear context entirely and start fresh |

Each mode confirms activation and tells you how to switch to the other.

---

## How it works

### Centralised prompts

The prompt instructions for each mode live in this repository, not in individual service
repos:

```
engineering-docs/
└── .claude/
    └── prompts/
        ├── docs-mode.md    ← prompt for /docs
        └── work-mode.md    ← prompt for /work
```

**Files:** `.claude/prompts/docs-mode.md` and `.claude/prompts/work-mode.md`

### Service repo commands

Each service repo contains two thin command files that are pointers — they instruct
Claude to fetch the prompt from engineering-docs via the GitHub MCP server and follow it:

```
your-service-repo/
└── .claude/
    └── commands/
        ├── docs.md    ← fetches docs-mode.md from engineering-docs
        └── work.md    ← fetches work-mode.md from engineering-docs
```

**`docs.md` content:**
```
Using the GitHub MCP server, read the file `.claude/prompts/docs-mode.md`
from the rawsharklives/engineering-docs repository and follow the instructions
in it exactly.
```

**`work.md` content:**
```
Using the GitHub MCP server, read the file `.claude/prompts/work-mode.md`
from the rawsharklives/engineering-docs repository and follow the instructions
in it exactly.
```

### Why this approach

Service repo command files are static and never need to change. When the prompt behaviour
needs updating — for example, to point at a different docs repo, tighten the mode
restrictions, or change the confirmation message — you update the prompt files in
engineering-docs once. Every service repo picks up the change immediately on next
invocation via MCP, with no PRs required in individual repos.

```
Update prompt in engineering-docs
        ↓
All service repos use new behaviour immediately
        ↓
No service repo PRs needed
```

---

## Setting up a new service repo

### Prerequisites

- GitHub MCP server configured in Claude Code (`claude mcp list` should show github as connected)
- Claude Code installed locally

### Steps

**1. Create the commands directory**

```bash
mkdir -p .claude/commands
```

**2. Create `.claude/commands/docs.md`**

```
Using the GitHub MCP server, read the file `.claude/prompts/docs-mode.md`
from the rawsharklives/engineering-docs repository and follow the instructions
in it exactly.
```

**3. Create `.claude/commands/work.md`**

```
Using the GitHub MCP server, read the file `.claude/prompts/work-mode.md`
from the rawsharklives/engineering-docs repository and follow the instructions
in it exactly.
```

**4. Commit and push**

```bash
git add .claude/commands/
git commit -m "Add /docs and /work Claude Code mode commands"
git push
```

That's it. The commands are immediately available in any Claude Code session opened in
that repo.

### Verification

Open a Claude Code session in the repo and run `/docs`. Claude should:
1. Fetch `docs-mode.md` from engineering-docs via MCP
2. Confirm it is in Docs Mode
3. Tell you to type `/work` to switch back

If it fails, check the MCP server is connected:
```bash
claude mcp list
```

---

## Updating mode behaviour

To change what `/docs` or `/work` does for all repos:

1. Edit `.claude/prompts/docs-mode.md` or `.claude/prompts/work-mode.md` in this repo
2. Raise a PR and merge to `main`
3. All service repos immediately use the updated behaviour — no changes needed elsewhere

---

## Reference

| Item | Location |
|------|----------|
| Docs mode prompt | `engineering-docs/.claude/prompts/docs-mode.md` |
| Work mode prompt | `engineering-docs/.claude/prompts/work-mode.md` |
| Example service command files | `example-service/.claude/commands/docs.md` / `work.md` |
| MCP server setup guide | `content/onboarding/mcp-setup.md` *(to be written)* |
