---
title: "ADR-001 Addendum A — GitHub Copilot + GHEC Variant"
date: 2026-04-21
author: Rich — VP Engineering
reviewers: Platform
status: Proposed
hideFromList: true
---

## Purpose

The core ADR presumes Claude Code as the AI runtime on engineer machines, with Anthropic's GitHub MCP server as the bridge to `engineering-docs`. This addendum documents what changes when the AI runtime is GitHub Copilot (CLI, IDE, and cloud agent) — **the same GHEC repository, site, and content remain**; only the engineer-facing AI layer, custom-instructions format, and auth plumbing change.

This is viable today because Copilot now supports Anthropic's Claude models natively — administrator-enabled via Copilot policy, no separate Anthropic billing, no BYO API key required at the user level. The underlying intelligence for `/docs`-style workflows is therefore unchanged; only the host wrapping it differs.

## Scope of changes

Only the layers below shift. Content, Hugo, CODEOWNERS, GitHub Pages, templates, diagramming, and access control carry over without modification.

| Layer | Core ADR (Claude Code) | This addendum (Copilot + GHEC) |
|---|---|---|
| Engineer IDE runtime | Claude Code CLI / Claude Desktop | GitHub Copilot in VS Code / Visual Studio / JetBrains / Copilot CLI |
| Underlying model | Claude (direct) | Copilot-hosted Claude Sonnet 4.6 / Opus 4.7 (policy-enabled) |
| Global instructions | `~/.claude/CLAUDE.md` | `$HOME/.copilot/copilot-instructions.md` (Copilot CLI) plus VS Code user profile instructions |
| Repo-local instructions | `CLAUDE.md` in service repo | `AGENTS.md` (primary) and `.github/copilot-instructions.md` — both supported; `CLAUDE.md` also read |
| Slash commands (`/docs`, `/work`) | `.claude/commands/*.md` | `.github/prompts/*.prompt.md` in IDE; **not currently supported in Copilot CLI** |
| MCP host | Claude Code mcp config | `.vscode/mcp.json`, Copilot CLI `~/.copilot/mcp-config.json`, IDE MCP registry |
| MCP auth | Custom Entra broker → GitHub App installation token | **Native OAuth** against GitHub (Entra JIT still usable via EMU / Entra-federated login) |
| Async agent | N/A (Claude Code is synchronous) | **Copilot cloud agent** (optional) — assign GitHub issues to Copilot to work autonomously against `engineering-docs` |

## 1. AI model layer

Copilot administrators enable Claude models via organisation policy ([Managing policies and features for GitHub Copilot in your enterprise](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-policies-and-features)). Once enabled, engineers select the model in the Copilot Chat model picker in VS Code, Visual Studio, JetBrains IDEs, Copilot CLI (`/model`), and the Copilot cloud agent model dropdown on github.com. Available Anthropic models at time of writing include Claude Sonnet 4.6 (default for most tasks) and Claude Opus 4.7 (promotional 7.5x premium request multiplier until 30 April 2026 — worth noting in cost forecasting). GitHub maintains a zero data retention agreement with Anthropic for GA features ([Hosting of models for GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/model-hosting)), which simplifies our FCA/data-residency conversation relative to a direct Anthropic contract.

**Net:** the intelligence behind the `/docs` workflow is the same model family either way. The choice between Claude Code and Copilot is a host-runtime and governance choice, not a model-capability choice.

## 2. Custom instruction layer

Copilot reads a different set of files, but the conceptual split (global context, service-repo context) is identical. Per GitHub's [custom instructions reference](https://docs.github.com/en/copilot/reference/custom-instructions-support), Copilot supports:

- **Repository-wide:** `.github/copilot-instructions.md` at repo root.
- **Agent instructions:** `AGENTS.md` at repo root (and nested — nearest in the directory tree takes precedence). When both `AGENTS.md` and `.github/copilot-instructions.md` are present, both are used; `AGENTS.md` at the root is treated as primary.
- **Path-specific:** `.github/instructions/**/*.instructions.md` with an `applyTo` frontmatter glob. Supported in VS Code, Visual Studio, and Copilot cloud agent.
- **Personal:** `$HOME/.copilot/copilot-instructions.md` (Copilot CLI) or the VS Code user profile.
- **Legacy compatibility:** Copilot also reads `CLAUDE.md` and `GEMINI.md` at the repo root if present — this is explicitly documented behaviour, which is a useful transition story if we pilot Claude Code first and migrate later.

### Mapping

| Core ADR file | Copilot equivalent | Notes |
|---|---|---|
| `~/.claude/CLAUDE.md` | `$HOME/.copilot/copilot-instructions.md` + VS Code user profile | Distributed via onboarding script as before |
| `engineering-docs/CLAUDE.md` | `engineering-docs/AGENTS.md` + `.github/copilot-instructions.md` | Both present; Copilot reads both |
| Service-repo `CLAUDE.md` | Service-repo `AGENTS.md` (root) | Keep minimal — local build/test commands plus a pointer to the service page in `engineering-docs` |

The 4,000-character limit on custom-instruction files for Copilot code review ([documented here](https://docs.github.com/en/copilot/tutorials/use-custom-instructions)) does **not** apply to Chat or the cloud agent, but it's a useful discipline: keep instruction files tight and push detail into path-specific files.

## 3. `/docs` and `/work` modes

Copilot's analogue of Claude Code's slash commands is **prompt files** ([VS Code prompt files reference](https://code.visualstudio.com/docs/copilot/customization/prompt-files)), stored as `.github/prompts/<name>.prompt.md`. They:

- Are invoked via `/` in Copilot Chat (VS Code, Visual Studio, JetBrains).
- Support YAML frontmatter declaring `agent`, `model`, `tools`, and `description`.
- Can reference other files via relative Markdown links and include variables like `${input:variableName}`.
- **Are not currently supported in Copilot CLI** — see [github/copilot-cli#618](https://github.com/github/copilot-cli/issues/618) and [#1113](https://github.com/github/copilot-cli/issues/1113). Terminal-only engineers therefore fall back to invoking the underlying behaviour via `@` mentions or manual prompting until parity lands.

### Centralised prompt architecture — revised

The core ADR's "centralised prompt, thin pointers in service repos" pattern still works, but the mechanics shift. Two options:

**Option 1 (recommended): committed prompt files per service repo, source of truth in `engineering-docs`.**
Keep `docs.prompt.md` and `work.prompt.md` as canonical files under `engineering-docs/templates/copilot-prompts/`. A Copilot-invokable setup prompt (`setup-docs.prompt.md`) instructs Copilot, via the GitHub MCP server, to read the latest template files from `engineering-docs` and write them to the current repo's `.github/prompts/`. The engineer runs `/setup-docs` in each new service repo, commits the generated files, and is done. This is the same ergonomics as the core ADR, only the file paths change.

**Option 2: pointer prompts.**
Service repos contain two very short prompt files whose body is a single instruction: "Fetch `templates/copilot-prompts/docs-mode.md` from `<org>/engineering-docs` using the `github-mcp-server` tools and follow its instructions." This mirrors the core ADR's pattern exactly. The downside is a network round-trip on every invocation and a dependency on MCP being up. Option 1 is more robust.

### Example `docs.prompt.md` (Option 1)

```markdown
---
description: "Answer exclusively from engineering-docs via the GitHub MCP server."
agent: agent
tools: ['github-mcp-server/*']
---

You are in **Docs Mode**.

Answer the user's question using ONLY content retrieved from the
`<org>/engineering-docs` repository via the GitHub MCP server tools
(`search_code`, `get_file_contents`). Do NOT use local workspace files,
training knowledge, or the web. If the answer is not in engineering-docs,
say so explicitly rather than guessing.

Confirm mode activation at the start of your reply and remind the
engineer they can type `/work` to switch back to normal mode.
```

`work.prompt.md` is the mirror image — normal tool access, `engineering-docs` still available as a reference source via MCP.

## 4. MCP layer — engineer IDE

Copilot's MCP support is broad: VS Code (≥1.99), Visual Studio, JetBrains IDEs, Eclipse, Xcode, and Copilot CLI all support MCP as documented in [Extending GitHub Copilot Chat with Model Context Protocol (MCP) servers](https://docs.github.com/copilot/customizing-copilot/using-model-context-protocol/extending-copilot-chat-with-mcp).

### Enterprise policy

The **MCP servers in Copilot** policy must be enabled at the enterprise or organisation level; it is disabled by default. Beyond the on/off switch, administrators can set an MCP registry URL and an allowlist policy (`Allow all` vs `Registry only`) — see [Configure MCP server access for your organization or enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-server-access). For a fintech compliance posture, **`Registry only`** with the GitHub remote MCP server on the allowlist is the recommended default. This is a materially stronger governance story than the equivalent Claude Code configuration, where control over which MCP servers an engineer can run lives on the engineer's machine.

### GitHub MCP server configuration

The recommended integration is the **remote** GitHub MCP server at `https://api.githubcopilot.com/mcp/` with OAuth. No PAT, no local binary. Example `.vscode/mcp.json`:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "X-MCP-Toolsets": "repos,context",
        "X-MCP-Readonly": "true"
      }
    }
  }
}
```

Two configuration details worth making explicit in the platform runbook:

- **Toolset scoping.** The remote server supports `X-MCP-Toolsets` and, more granularly, `X-MCP-Tools` headers to enable specific tools only ([GitHub MCP server remote configuration docs](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md), and the [December 2025 changelog](https://github.blog/changelog/2025-12-10-the-github-mcp-server-adds-support-for-tool-specific-configuration-and-more/) introducing per-tool configuration). For Docs Mode we only need `get_file_contents` and the repository search tools (`search_code`, `search_repositories`) — nothing else. This trims the tool-definition payload in every model call (see Addendum B on scaling).
- **Read-only.** `X-MCP-Readonly: true` (or appending `/readonly` to the URL) locks the session to read tools. Docs Mode should always run read-only; the only write path we want is the explicit "raise a doc PR" flow, which we can handle via a separate, non-readonly prompt file.

### GHEC with data residency

If we ever move to GHEC with data residency, the MCP URL becomes `https://copilot-api.<subdomain>.ghe.com/mcp` per [Configuring the GitHub MCP Server for GitHub Enterprise](https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp/enterprise-configuration). GitHub Enterprise **Server** is not relevant for us, but worth noting that GHES only supports the local MCP binary — remote is GHEC-only.

## 5. AuthN / AuthZ — what falls away

The core ADR's Entra → Azure Function broker → GitHub App architecture was designed around a specific weakness: each Claude Code user otherwise ends up with a long-lived PAT in plaintext in `~/.claude.json`. With Copilot's remote MCP server, **this problem largely disappears**:

- VS Code, Visual Studio, JetBrains, Eclipse and Xcode authenticate via one-click OAuth on behalf of the signed-in Copilot user. No PAT is stored locally.
- The OAuth flow inherits the organisation's GitHub OAuth app restrictions and — for Enterprise Managed Users federated with Entra — our Entra Conditional Access policies (MFA, compliant device, etc.) apply at the GitHub sign-in step. This is functionally equivalent to the bespoke broker, without our team having to build or operate it.
- Offboarding is handled by removing the user from the GitHub organisation, which invalidates their Copilot seat and therefore all MCP access derived from it.

**Recommendation:** drop the Azure Function token-broker follow-on work item from the core ADR if we commit to Copilot. Retain Entra as the identity provider (via GHEC EMU); retain the Engineering Entra group as the access-control mechanism (now mapped to a GitHub team with read access to `engineering-docs`). We keep the auditability and offboarding properties; we lose a ~50-line Azure Function and a Key Vault secret to rotate.

**Caveat — PAT fallback for Copilot CLI.** Copilot CLI's remote GitHub MCP server is built-in and authenticated by default, but some edge cases still require a PAT (notably Enterprise Managed Users where the administrator has explicitly enabled PAT usage). Where a PAT is genuinely required, the Entra-broker pattern from the core ADR remains the right architecture for those users. In practice this will be a minority.

## 6. New capability — Copilot cloud agent

Copilot cloud agent (the background agent that can be assigned a GitHub issue or mentioned in a PR) is a genuine addition over the Claude Code baseline. It is relevant to `engineering-docs` in two ways:

1. **"Generate/update docs" as an assignable task.** Create an issue on `engineering-docs` ("add an ADR for the payments rewrite", "refresh runbook for the reconciliation job"), assign to Copilot, select Claude Opus 4.7 or Sonnet 4.6 from the [cloud agent model picker](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/cloud-agent/changing-the-ai-model), and Copilot produces a draft PR. Our CODEOWNERS review is unchanged — the agent is simply another PR author.
2. **Repository-level MCP configuration.** Cloud agent MCP configuration lives at `Settings → Copilot → Cloud agent` on the repo (per [Extending GitHub Copilot cloud agent with the Model Context Protocol (MCP)](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/cloud-agent/extend-cloud-agent-with-mcp)). The GitHub MCP server is enabled by default with a read-only token scoped to the current repo. This is precisely what Docs Mode needs, with zero additional configuration.

**Known limitation.** Copilot cloud agent does **not** currently support remote MCP servers that use OAuth for authentication. This means if we ever want the cloud agent to reach across to an external system (Jira, Confluence, Azure DevOps), we need a token-based auth path or the OIDC federation pattern GitHub documents for Azure DevOps. For the pure `engineering-docs` scope, this is not a constraint — the built-in GitHub MCP server is token-authenticated automatically.

## 7. Consequences (delta from core ADR)

### Positive

- **Single vendor, single bill.** Copilot Enterprise covers IDE, CLI, and cloud agent; Anthropic models are accessed via the same contract. Procurement and FCA/third-party risk conversations are simpler.
- **Stronger enterprise policy surface.** MCP allowlist, model policies, and Conditional Access via GHEC EMU are administered centrally rather than per-engineer.
- **Reduced auth plumbing.** The Entra broker + Azure Function + Key Vault follow-on work disappears for the majority of engineers.
- **Adds Copilot cloud agent** as a way to produce doc PRs from a GitHub issue, which Claude Code does not natively offer.
- **Broader IDE coverage.** Visual Studio, JetBrains, Eclipse, and Xcode are first-class alongside VS Code.

### Negative

- **Copilot CLI parity gap.** Prompt-file slash commands are not supported in Copilot CLI. Any engineer who prefers a terminal-native workflow loses the ergonomic `/docs`/`/work` switching until GitHub ships that feature (tracked in the two issues linked above).
- **Model-routing opacity.** With "Auto" model selection, the specific model backing a given response is less transparent than Claude Code. For sensitive work (e.g. anything with regulatory implications in a PR description), engineers should explicitly select Claude Opus 4.7 or Sonnet 4.6.
- **Cloud agent OAuth-MCP limitation.** Restricts some future integration options for the async agent specifically.
- **Premium request accounting.** Claude Opus 4.7 carries a 7.5x multiplier (promotional until 30 April 2026) — worth flagging in cost reporting before engineers default to it for every query.
- **Tighter coupling to the GitHub ecosystem.** If we ever migrate off GHEC, the custom-instructions, prompt files, and MCP configuration are all GitHub-proprietary formats. The `CLAUDE.md` fallback eases this but doesn't eliminate it.

### Follow-on work (delta)

Add:

- Enable the `MCP servers in Copilot` policy at the enterprise level; set MCP access to `Registry only` with the GitHub remote MCP server allowlisted.
- Enable Claude Sonnet 4.6 and Claude Opus 4.7 policies in Copilot Enterprise settings.
- Add `AGENTS.md` and `.github/copilot-instructions.md` to `engineering-docs`, mirroring the existing `CLAUDE.md` content.
- Author `templates/copilot-prompts/docs-mode.prompt.md`, `work-mode.prompt.md`, and `setup-docs.prompt.md`.
- Write a onboarding script to drop `.vscode/mcp.json`, `$HOME/.copilot/copilot-instructions.md`, and (for CLI users) `~/.copilot/mcp-config.json` into the engineer's environment. This replaces the PowerShell-for-Claude-Desktop script in the core ADR.
- Decide cloud agent policy: which repos can assign issues to Copilot, which cannot. Recommend enabling for `engineering-docs` itself as a docs-generation workflow; disable by default for service repos until we have experience.

Remove (or defer):

- Entra Enterprise App `engineering-docs.read` role (kept at GHEC/Entra sign-in layer instead).
- Azure Function token broker.
- Azure Key Vault entry for the GitHub App private key.
- PowerShell onboarding script for Claude Desktop.

## 8. Recommendation

Proceed with the Copilot + GHEC variant if the team's primary IDE is VS Code, Visual Studio, or JetBrains, and the majority of engineer AI usage is inside the IDE. Revert to the core ADR (Claude Code) only if a material proportion of engineers live in the terminal or need features Copilot CLI does not yet support (structured mode switching being the main gap today).

Either way, **the content, the Hugo site, access control, diagrams, and the `engineering-docs` repository layout are unchanged**. Switching between the two is a matter of swapping the AI runtime and auth story on top of the same knowledge base — which is itself a useful property and one of the arguments for the docs-as-code decision in the first place.

## References

- [Hosting of models for GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/model-hosting)
- [Supported AI models in GitHub Copilot](https://docs.github.com/en/copilot/reference/ai-models/supported-models)
- [Support for different types of custom instructions](https://docs.github.com/en/copilot/reference/custom-instructions-support)
- [Adding repository custom instructions for GitHub Copilot](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
- [Use prompt files in VS Code](https://code.visualstudio.com/docs/copilot/customization/prompt-files)
- [Extending GitHub Copilot Chat with MCP servers](https://docs.github.com/copilot/customizing-copilot/using-model-context-protocol/extending-copilot-chat-with-mcp)
- [Configure MCP server access for your organization or enterprise](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-mcp-usage/configure-mcp-server-access)
- [Configuring the GitHub MCP Server for GitHub Enterprise](https://docs.github.com/en/copilot/how-tos/provide-context/use-mcp/enterprise-configuration)
- [github/github-mcp-server — remote server configuration](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [Extending GitHub Copilot cloud agent with MCP (GHEC)](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/cloud-agent/extend-cloud-agent-with-mcp)
- [Changing the AI model for Copilot cloud agent](https://docs.github.com/en/enterprise-cloud@latest/copilot/how-tos/use-copilot-agents/cloud-agent/changing-the-ai-model)
