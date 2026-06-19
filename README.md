# IAM + AI Wiki

A living, LLM-maintained knowledge base on the collision between **Identity & Access Management** and **AI agents** — agentic identity, MCP authorization, delegated authority, confused-deputy attacks, prompt injection as auth bypass, and the rest of the fast-moving frontier where OAuth meets autonomous agents.

Its distinguishing lean: **every page bridges back to traditional enterprise IAM.** New agentic concepts are mapped to the specific RFCs, grant types, and patterns a seasoned IAM practitioner already knows — then the page explains exactly where that pre-AI model breaks down.

> _"An MCP client is, almost by definition, a [confused deputy](wiki/concepts/confused-deputy.md) — a deputy that wields delegated authority across many tools and servers."_

## What this is (and isn't)

This is an instantiation of **Andrej Karpathy's LLM Wiki** concept — the full idea is captured in [`llm-wiki-spec.md`](llm-wiki-spec.md), sourced from his [original Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

The core distinction from RAG: a RAG system re-discovers knowledge from scratch on every query. An LLM Wiki **compiles knowledge once and keeps it current.** When a new source arrives, the LLM reads it, integrates it into existing pages, updates cross-references, and flags contradictions. The wiki is a *persistent, compounding artifact* — the synthesis already reflects everything read so far.

> You curate sources, explore, and ask good questions. The LLM does all the grunt work — summarizing, cross-referencing, filing, and bookkeeping. You read the wiki; you (almost) never write it.

In practice: the LLM agent on one side, [Obsidian](https://obsidian.md) open on the other. The agent edits; you browse the graph in real time.

## Layout

Three layers — **immutable sources**, an **LLM-owned wiki**, and a **schema** ([`CLAUDE.md`](CLAUDE.md)) that makes the LLM a disciplined maintainer rather than a generic chatbot.

```
raw/           # immutable source documents — the LLM reads, never writes (third-party; see raw/README.md)
wiki/          # curated, interlinked pages — owned entirely by the LLM
  concepts/    # foundational ideas (agentic-identity, confused-deputy, …)
  topics/      # subjects w/ active discourse (mcp-authorization, scope-selection, …)
  references/  # specs, RFCs, drafts, tools (rfc-7636-pkce, oauth-2-1, …)
  entities/    # vendors, orgs, working groups (anthropic, ietf-oauth-wg, …)
  incidents/   # real-world auth-bypass / credential-leak events
  sources/     # one page per ingested raw doc, summarizing its claims
_index.md      # catalog of every page, grouped by category
log.md         # append-only chronological record of every change
```

## Every page is structured

Wikilinks (`[[slug]]`) connect pages; each carries YAML frontmatter so the graph is queryable (e.g. via Obsidian Dataview):

```yaml
---
title: Confused Deputy
category: concept          # concept | topic | reference | entity | incident | source
status: stable             # stub | stable | evolving | contested | deprecated | superseded
confidence: high           # high | medium | low — tiered by source authority (RFC > vendor blog > social)
aliases: [confused deputy problem, deputy confusion]
enterprise_analogs: [OAuth 2.0 mix-up attack, CSRF, RFC 9700 Security BCP, ambient authority]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations]
related: [token-passthrough, authorization-server-mix-up, mcp-authorization]
tags: [security, confused-deputy, threat-model]
---
```

The `enterprise_analogs` field — plus two prose sections, **"Relation to pre-AI IAM"** and **"Why pre-AI IAM is insufficient"** — are required on every concept and topic. That's the bridge.

## The three workflows

The LLM operates the wiki through three repeatable workflows, each defined in [`CLAUDE.md`](CLAUDE.md). Every one of them ends by updating `_index.md` and appending to `log.md`.

| Workflow | Trigger | What the LLM does |
|----------|---------|-------------------|
| **Ingest** | A new doc lands in `raw/` | Summarize it into `wiki/sources/`, then create/update every concept, topic, reference, entity, and incident it touches — cross-referencing as it goes. One source can touch 10–15 pages. |
| **Query** | You ask a question | Read `_index.md` to locate pages, read them in full, synthesize a cited answer. Genuinely new synthesis gets **filed back** as a wiki page, so explorations compound. |
| **Lint** | Periodic health check | Hunt for contradictions, stale claims, orphan pages, missing pages, broken links, and coverage gaps against the in-scope list — then fix them. |

## Two navigation files

- **`_index.md`** — content-oriented catalog. One line per page (`[[slug]] — summary · status · confidence`), grouped by category. The LLM reads it *first* on every query to find relevant pages without embedding-based RAG.
- **`log.md`** — chronological, append-only. Each entry is prefixed `## [YYYY-MM-DD] <workflow> | <title>`, so the timeline is greppable:

  ```sh
  grep '^## \[' log.md | tail -5
  ```

## Scope

**In:** agentic identity · machine/workload identity for agents · MCP authN/authZ · OAuth/OIDC adaptations for LLM agents · identity propagation in agent chains · prompt injection as auth bypass · delegated authorization · tool-use scoping · confused-deputy patterns · SPIFFE/SPIRE for AI workloads · human-in-the-loop authorization.

**Out:** general AI safety · prompt engineering · RAG architecture (except where it touches auth) · LLM training methodology.

## Status

Seeded from the **MCP Authorization draft specification** (all four documents ingested). Browse [`_index.md`](_index.md) for the current page catalog and [`log.md`](log.md) for the change history. The wiki is just a git repo of markdown — you get version history, branching, and the Obsidian graph view for free.
