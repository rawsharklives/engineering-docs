---
title: "ADR-001 Addendum B — Scaling the Documentation MCP"
date: 2026-04-21
author: Rich — VP Engineering
reviewers: Platform
status: Proposed
hideFromList: true
---

## Purpose

The core ADR's MCP integration treats `engineering-docs` as a repository the AI browses on demand. That is the right starting point at tens-to-low-hundreds of pages. This addendum examines what happens as the docs store grows to thousands of pages, identifies the specific bottlenecks that appear before it does, and sets out a staged progression toward semantic retrieval.

The short version: at our current scale the GitHub MCP server is fine. At ~500 pages we should add **measurement and tool-surface discipline**. At ~1,000+ pages we should move to a **RAG layer with a vector index** fronted by our own MCP server. We should not build any of that until the metrics say we need it.

## How the current MCP integration actually works

Two things happen in every Copilot/Claude conversation that uses the GitHub MCP server, and both consume context-window tokens:

1. **Tool-definition payload (per session / per request).** When the client starts an MCP session it calls `tools/list` against `https://api.githubcopilot.com/mcp/`, and the server returns JSON schemas for every tool in the enabled toolsets. Those schemas are then included in the system prompt on **every request** in that session. The GitHub remote MCP server exposes roughly 70+ tools across all toolsets today; with default toolsets enabled (`context`, `repos`, `issues`, `pull_requests`) this is a meaningful baseline tax before the engineer has typed anything. Industry measurement suggests tool metadata can consume 40–50% of available context at the high end ([The New Stack, Feb 2026](https://thenewstack.io/how-to-reduce-mcp-token-bloat/)), and third-party analysis ([MCP Playground, 2026](https://mcpplaygroundonline.com/blog/mcp-token-counter-optimize-context-window)) puts a 50-tool MCP server at roughly 10,000–20,000 tokens of schema overhead per request.

2. **Tool-call results (per invocation).** When Copilot calls `search_code` or `get_file_contents`, the returned content is injected into the conversation. `get_file_contents` on a page returns the whole Markdown file; `search_code` returns matched snippets against GitHub's code-search backend.

The first cost is constant in the size of the docs store but proportional to the number of tools enabled. **The second cost is where the docs-scale problem actually lives.**

## What actually breaks at scale

### Bottleneck 1 — No semantics in `search_code`

GitHub's code search (the backend behind `search_code`) is a **lexical** search — qualifiers, exact matching, some tokenisation, but no concept of semantic similarity. An engineer asking "how do we handle failover for the reconciliation job?" will not surface a runbook titled "Reconciliation service — disaster recovery procedure" unless the word "failover" is in that file. At 50 pages this is tolerable because the model can browse; at 2,000 pages the model cannot usefully browse and the retrieval misses will manifest as Docs Mode saying "I can't find this in engineering-docs" when the answer is demonstrably there.

### Bottleneck 2 — Context-window consumption per page retrieved

A dense runbook is typically 500–2,000 tokens in Markdown. Copilot (and Claude Code) will often call `get_file_contents` on 3–10 candidate files to reason about a question. At 10 files × 1,500 tokens = 15,000 tokens injected per turn, on top of the ~10,000-token MCP schema overhead. Claude Sonnet 4.6 and Opus 4.7 handle this comfortably in absolute terms, but it has three second-order effects:

- **Cost.** Every retrieved page is billed on every turn it remains in context. At high docs volume and high engineer usage this compounds. Prompt caching ([Anthropic's cache](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)) helps for repeated sessions with the same tool definitions but does not help for freshly retrieved page content.
- **Latency.** Each tool call is a round-trip; chained retrieval (find the ADR → find the linked runbook → find the service page) is serial.
- **Quality degradation on "read the whole thing" queries.** Asking "summarise our platform architecture" at 2,000 pages is infeasible as a plain-MCP call.

### Bottleneck 3 — Rate and search-result limits

GitHub's code search API returns a capped number of results per query (page size max 100, with practical quality degradation well before that) and is subject to authenticated rate limits. None of this is a blocker at low volume, but both become problematic with many engineers making many retrievals per day.

### What does **not** break

Hugo build time, GitHub Pages delivery, git performance on the monorepo, and the Fuse.js → Pagefind migration path for the human-facing site are all unaffected by AI retrieval scaling. Those are separate concerns and the core ADR addresses them correctly. This addendum is strictly about the AI retrieval path.

## Scaling progression

Treat this as a three-stage ramp, driven by measured pain rather than page count alone. The triggers are concrete and cheap to observe.

### Stage 1 — Current state, with discipline (< ~500 pages)

**No architectural change.** The GitHub MCP server is adequate. Two tightenings that materially improve behaviour at this stage and compound benefit at every later stage:

- **Constrain the tool surface.** Docs Mode only needs `get_file_contents`, `search_code`, and possibly `list_commits`. Use the `X-MCP-Tools` header (per the [December 2025 GitHub MCP changelog](https://github.blog/changelog/2025-12-10-the-github-mcp-server-adds-support-for-tool-specific-configuration-and-more/)) rather than enabling entire toolsets:

  ```json
  {
    "servers": {
      "github-docs": {
        "type": "http",
        "url": "https://api.githubcopilot.com/mcp/",
        "headers": {
          "X-MCP-Tools": "get_file_contents,search_code",
          "X-MCP-Readonly": "true"
        }
      }
    }
  }
  ```

  This trims the schema payload from ~70 tools to 2, recovering a significant slice of the context window for actual reasoning.

- **Author for retrieval.** Treat every doc's first paragraph as the thing a search-based retriever will match on. Include the canonical name, the service boundary, and the problem it solves in the first 200 characters. Add a concise `summary:` frontmatter field. This is essentially free and makes every later retrieval strategy more effective.

**Trigger to advance to Stage 2:** total Markdown in `content/` exceeds ~1–2 MB (measurable in CI), or engineers start reporting Docs Mode misses on questions whose answers clearly exist in the repo.

### Stage 2 — Hybrid with a docs index (~500–1,500 pages)

Introduce a **lightweight local index** that sits alongside the GitHub MCP server, still without a vector database. Two realistic options:

- **Pagefind, reused.** The core ADR adopts Pagefind for the human-facing site. Pagefind's chunked index is also queryable programmatically — we can expose it as a small MCP tool (`search_docs` returning URL + excerpt) via a thin MCP server running in `engineering-docs` itself, deployed alongside the Hugo site. This gives us BM25-style full-text search over the rendered site for the cost of one small repo. Zero new infrastructure.
- **GitHub's own semantic tooling.** The remote GitHub MCP server has been gaining additional toolsets, including the in-beta `github_support_docs_search` ([per the server README](https://github.com/github/github-mcp-server)). Watch for a general-purpose `docs_search` toolset — if GitHub ships one backed by an organisation-level docs index (as Copilot Spaces hints at), we inherit it for free.

At this stage we are still not vectorising anything ourselves. We are improving the **retrieval quality** of the search the model performs, without changing the transport (MCP) or the source of truth (`engineering-docs`). The model still reads full files via `get_file_contents`; the `search_docs` tool just makes it more likely to pick the *right* files.

**Trigger to advance to Stage 3:** lexical search misses start to dominate the failure mode (engineers know the answer exists, Docs Mode can't find it), or we cross ~1,500 pages, or specific high-value use cases (onboarding chatbot, incident response agent) need sub-second grounding over the whole corpus.

### Stage 3 — Vector RAG, fronted by a custom MCP server (~1,500+ pages)

At this point we accept the operational cost of a real retrieval layer. The target architecture:

```
engineer → Copilot/Claude → Custom "docs" MCP server
                              ↓
                  Azure AI Search (hybrid: BM25 + vector + semantic ranker)
                              ↑
        Indexer pulling from engineering-docs on every merge to main
```

**Why Azure AI Search specifically.** We already live in the Microsoft estate — Entra, Key Vault, and Conditional Access are set up; GHEC EMU federates against the same tenant; Netwealth's compliance posture is already documented against Azure services. Azure AI Search ([Microsoft Learn — RAG overview](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)) provides:

- Managed indexer with integrated chunking and vectorisation — we don't run embedding pipelines ourselves.
- Hybrid queries combining BM25 and vector similarity via Reciprocal Rank Fusion (RRF) — empirically better than either alone ([Vector search overview](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)).
- A semantic ranker (L2) that rescores top results using a language model — measurably lifts answer quality for RAG.
- **Agentic retrieval** ([introduction](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)), where the service itself decomposes a question into sub-queries, runs them in parallel, and returns a unified grounded response. This maps directly onto what Docs Mode does today via serial MCP calls, but faster and at lower token cost.
- Data residency and the usual Azure compliance certifications.

The alternatives (self-hosted Qdrant/Weaviate/Milvus; Pinecone; pgvector on an existing Postgres) are all reasonable in principle but add operational surface area — embedding pipelines, vector DB ops, index lifecycle — that Azure AI Search manages. For a small team, managed is the right default.

**The custom docs MCP server.** A small service (Go, Python, or Node — ~200–400 lines of actual logic) exposing a single, focused tool:

```
search_engineering_docs(query: string, k: int = 5) →
  [{ path, title, url, score, excerpt }]
```

It calls Azure AI Search, returns the top-k chunks with enough context for the model to decide which ones to deepen (via `get_file_contents` on the GitHub MCP server, or by following the returned URL). Copilot's Registry-only MCP policy allowlists this server in addition to the GitHub one.

**Why keep the GitHub MCP server at all?** Because the model still needs the ability to read *whole* files when it decides to, raise PRs, and interact with issues — things that are repository actions, not search. The pattern is **complementary, not replacement**: Azure AI Search is the retrieval brain; GitHub MCP is the action surface. This is exactly the pattern Speakeasy describes in [MCP vs RAG](https://www.speakeasy.com/blog/rag-vs-mcp) — RAG for efficient grounding, MCP for standardised tool access.

**Indexing.** An Azure AI Search indexer pulls from `engineering-docs` — either directly from the GitHub repository, or from a Blob Storage mirror synced by the existing `deploy.yml` workflow. A skillset handles chunking and embedding (Azure OpenAI `text-embedding-3-large` or equivalent). Reindexing is incremental and triggered on every merge to `main`. The authoritative source of truth remains the Markdown in Git; the index is a derived, disposable artefact.

**Auth.** The custom MCP server authenticates the caller the same way our other Azure services do — Entra JWT validated at the function front door, with the Engineering group scoped to read access on the index. Managed identity on the function reads the Azure AI Search admin key from Key Vault. This is the same pattern the core ADR already sketches for the GitHub App token broker; we reuse it.

### Summary table

| Stage | Trigger | Change | Infra cost | Operational cost |
|---|---|---|---|---|
| 1 | Today | Discipline: tool-surface trimming, better authoring | None | ~zero |
| 2 | ~500 pages, or `content/` > 1–2 MB | Add Pagefind-backed `search_docs` MCP tool OR adopt GitHub's native docs search toolset when GA | None — uses existing Hugo/Pages stack, or GitHub-provided | Low — one small MCP server to deploy |
| 3 | ~1,500 pages, lexical misses dominate, or high-throughput agent use cases | Azure AI Search + custom docs MCP server | Azure AI Search (Basic tier ~£70/mo entry, scales) + embeddings API | Moderate — indexer lifecycle, embedding cost, Conditional Access on the MCP endpoint |

## What I am **not** recommending

- **Building RAG now.** The cost of premature vectorisation — operational complexity, index staleness bugs, embedding-cost creep — dwarfs the benefit when the MCP baseline is still working. We should know what we're solving before we solve it.
- **Replacing the GitHub MCP server.** It remains the action surface even after we add Azure AI Search. Authoring a PR, reading a specific file, browsing commits — all still go through it.
- **Embedding code.** This addendum is about docs. If and when we want semantic search over service code, that is a separate decision and GitHub's own `github_support_docs_search` / Copilot Spaces work is the most likely answer there rather than rolling our own.
- **Exposing the vector index to engineers directly.** The index is an implementation detail. The contract with engineers — and with Copilot/Claude — is the MCP tool surface. That means we can swap Azure AI Search for something else later without any engineer-facing change.

## Consequences

### Positive

- **Measurement-led scaling.** Each stage is triggered by a concrete observable, not a guess about future volume.
- **Tool-surface discipline helps immediately.** Even the Stage 1 changes (`X-MCP-Tools` scoping, frontmatter summaries) improve quality and reduce cost today.
- **Clean separation.** The MCP contract is stable across all three stages. Engineers see the same `/docs` experience; only the retrieval backend changes.
- **Plays to our Microsoft estate.** Stage 3's Azure AI Search choice reuses Entra, Key Vault, and our existing compliance posture.

### Negative

- **Stage 3 introduces a real operational burden.** Indexer lifecycle, embedding cost, Conditional Access scoping, MCP server deployment, index-staleness monitoring — none of this is free.
- **Azure AI Search has a minimum cost floor** even at Basic tier. Below the usage threshold where RAG is justified, it's pure overhead.
- **Vendor coupling increases at Stage 3.** Azure AI Search is not portable in the way that Markdown-in-Git is. This is acceptable precisely because the source of truth remains portable; the index is derived.
- **More moving parts means more things to monitor.** A stale index that silently returns outdated search results is a worse failure mode than a search-miss, because Docs Mode will confidently return a wrong answer.

### Follow-on work

**Stage 1 (do now, regardless of which variant of the core ADR we adopt):**

- Update the Docs Mode MCP configuration to use `X-MCP-Tools` with a minimal tool set.
- Add a `summary:` frontmatter field and first-paragraph convention to the doc templates in `templates/`.
- Add a CI check that reports the total size of `content/` on each PR, so we can see the growth curve.

**Stage 2 (when triggered):**

- POC a Pagefind-backed MCP `search_docs` tool, or adopt GitHub's docs-search toolset if it GAs in a suitable form.
- Re-measure Docs Mode miss rate before and after.

**Stage 3 (when triggered):**

- POC Azure AI Search with a subset of `engineering-docs` content (agentic retrieval quickstart: [Microsoft Learn](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)).
- Build the custom docs MCP server and register it in the Copilot MCP registry.
- Define reindexing cadence and staleness monitoring.
- Update the enterprise MCP allowlist to include the custom server.

## References

- [RAG and Generative AI — Azure AI Search (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/retrieval-augmented-generation-overview)
- [Vector Search Overview — Azure AI Search (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/vector-search-overview)
- [Introduction to Azure AI Search (Microsoft Learn)](https://learn.microsoft.com/en-us/azure/search/search-what-is-azure-search)
- [The GitHub MCP Server adds support for tool-specific configuration (GitHub Changelog, Dec 2025)](https://github.blog/changelog/2025-12-10-the-github-mcp-server-adds-support-for-tool-specific-configuration-and-more/)
- [github/github-mcp-server — remote server configuration](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [MCP vs RAG (Speakeasy, 2025)](https://www.speakeasy.com/blog/rag-vs-mcp)
- [10 strategies to reduce MCP token bloat (The New Stack, Feb 2026)](https://thenewstack.io/how-to-reduce-mcp-token-bloat/)
- [MCP Token Counter (MCP Playground, 2026)](https://mcpplaygroundonline.com/blog/mcp-token-counter-optimize-context-window)
- [Anthropic prompt caching documentation](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching)
