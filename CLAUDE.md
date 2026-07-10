# IAM + AI Wiki — Operating Schema

## Scope

**In-scope:**
- Agentic identity
- Machine identity for AI agents
- MCP authentication and authorization
- OAuth/OIDC adaptations for LLM agents
- Identity propagation in agent chains
- Prompt injection as auth bypass
- Delegated authorization for AI
- Attestation for AI-generated tokens
- Agent-to-agent authentication
- Tool-use authorization and scoping
- Confused-deputy patterns in agentic contexts
- Workload identity for AI workloads (e.g., SPIFFE/SPIRE)
- Human-in-the-loop authorization

**Out-of-scope:**
- General AI safety
- Prompt engineering
- RAG architecture except where it touches auth
- LLM training methodology

## Layout

- `raw/` — immutable source documents. Never modified.
- `wiki/` — curated pages. Owned by Claude.
  - `wiki/concepts/` — foundational ideas
  - `wiki/topics/` — specific subjects with active discourse
  - `wiki/references/` — specs, RFCs, drafts, tools, frameworks
  - `wiki/entities/` — vendors, products, organizations, working groups
  - `wiki/incidents/` — specific real-world incidents (e.g., prompt-injection-driven auth bypasses, agent credential leaks)
  - `wiki/sources/` — one page per ingested raw doc, summarizing its claims
- `_index.md` — catalog of every wiki page, grouped by category
- `log.md` — append-only chronological record

## Naming

- Files: kebab-case (e.g., `agentic-identity.md`)
- Wikilinks use the slug: `[[agentic-identity]]`
- Human-readable name lives in the `title` frontmatter field
- Alternate terms go in `aliases` frontmatter

## Frontmatter (required on every wiki page)

```yaml
---
title: <human-readable name>
category: <concept | topic | reference | entity | incident | source>
status: <stub | stable | evolving | contested | deprecated | superseded>
confidence: <high | medium | low>
aliases: ["<alternate term>", "<alternate term>"]
enterprise_analogs: ["<pre-AI IAM patterns this maps to; required on concepts/topics, conditional elsewhere — see Bridging>"]
last_updated: <YYYY-MM-DD>
sources: ["<wiki/sources/ slug or raw/ file path>"]
related: ["<wiki page slug>"]
tags: ["<tag>"]
---
```

Every item in every frontmatter list is double-quoted — see *Frontmatter list quoting* under Conventions. An empty list is written `[]`.

## Confidence rules

Override the generic rule with these tiers for this domain.

**High:**
- IETF RFCs and active OAuth/OIDC WG drafts
- MCP specification (modelcontextprotocol.io)
- NIST publications (e.g., AI RMF)
- OWASP LLM Top 10
- OpenID Foundation specs and drafts (FAPI, AuthZEN, etc.)
- SPIFFE/SPIRE specifications
- Peer-reviewed papers

**Medium:**
- Vendor engineering blogs (Anthropic, OpenAI, Okta, Auth0, Microsoft Entra, Google Cloud IAM, AWS IAM, Cloudflare)

**Low–medium:**
- Everything else (Substack, conference talks, podcasts, social posts)

## Workflows

### General
- Never modify files under `raw/`
- All wiki edits land under `wiki/`
- Every change updates `_index.md` and appends to `log.md`

### Ingest
1. Read the source in `raw/`
2. Create or update `wiki/sources/<slug>.md` summarizing its claims, citing the raw doc
3. Identify the entities, concepts, topics, references, and incidents it covers
4. Update or create the affected pages under `wiki/concepts/`, `wiki/topics/`, `wiki/references/`, `wiki/entities/`, `wiki/incidents/`
5. Add cross-references to the new source page from every page it touches
6. Update `_index.md`
7. Append to `log.md`

### Query
1. Read `_index.md` first to locate relevant pages
2. Read the relevant pages in full
3. Synthesize an answer with citations to wiki pages and underlying raw sources
4. If the synthesis discovered new or useful content, file it back as a new wiki page; update `_index.md` and `log.md`

### Lint
1. Health-check the wiki for:
   - Contradictions between pages
   - Stale claims superseded by newer sources
   - Orphan pages (no inbound links)
   - Important concepts mentioned but lacking their own page
   - Stub pages awaiting research
   - Missing or broken cross-references
   - Gaps in coverage relative to the in-scope list
   - Pages missing the bridging sections where defaults call for them
   - Frontmatter list items that are not double-quoted, and any item silently split or coerced by an unquoted `,` or `: ` (see *Frontmatter list quoting*)
2. Apply fixes (update pages, add stubs, repair links, adjust `status`)
3. Update `_index.md` and append to `log.md`

## Conventions

- **Wikilinks + standard markdown.** Use `[[slug]]` for internal links.
- **Never hard-wrap prose.** Write each paragraph and each list item as a single continuous line and rely on the editor's soft-wrap (Obsidian, VS Code, GitHub). Do not insert fixed-column line breaks inside `.md` files, and do not use trailing-two-space or trailing-backslash hard breaks. Headings, tables, code fences, and the blank lines between blocks stay on their own lines as usual. This applies to every markdown file the wiki owns — `wiki/`, `_index.md`, `log.md`, `README.md`, and `CLAUDE.md` itself — but never to `raw/`, which is immutable and keeps whatever wrapping its source had. It governs Markdown only: git commit messages are hard-wrapped as usual.
- **Frontmatter list quoting.** Every item in every frontmatter list (`aliases`, `enterprise_analogs`, `sources`, `related`, `tags`) MUST be wrapped in double quotes, including bare kebab-case slugs: `related: ["mcp-authorization", "token-theft"]`. Write an empty list as `[]`. The rule is unconditional so that no author has to notice the hazard: inside a YAML flow sequence a bare `,` is an item separator and a bare `: ` is the mapping indicator, so an unquoted `OS elevation prompts (UAC, Gatekeeper)` silently becomes two list items and an unquoted `DOM-based XSS via javascript: URIs` silently becomes a nested dictionary. Both corruptions are invisible in the source text. Quoting is also a prerequisite for ever using wikilinks as property values, since a bare `[[slug]]` parses as a nested sequence rather than a link. Double-quoted style makes `\` an escape character: an item containing a backslash or a literal `"` must escape it as `\\` or `\"`. This applies to `wiki/` pages only; `raw/` is immutable.
- **Citations.** Every non-trivial claim cites either a `wiki/sources/<slug>.md` page or a `raw/` file path.
- **Contested vs superseded.**
  - *Contested*: two current sources actively disagree. Surface both under a "Contested claims" section on the relevant page. Set `status: contested`.
  - *Superseded*: a newer source clearly replaces an older claim. Set the older page's `status: superseded` and link to the newer one.
- **Voice.** Encyclopedic-neutral. No marketing language. Attribute vendor-specific terminology explicitly (e.g., "Anthropic refers to X as Y; the equivalent OAuth term is Z").
- **Terminology.** This field uses overlapping, fast-evolving vocabulary. Prefer the most-cited or spec-defined term as canonical; list others in `aliases`.
- **Stub pages.** When a page references a concept/entity/incident that warrants its own page but no source yet supports it, create the target page with `status: stub` and a one-line description. Lint surfaces stubs as gaps to fill.

## Bridging to pre-AI IAM

The reader has deep pre-AI IAM background (OAuth2, OIDC, SAML, SCIM, classic AuthN/AuthZ). Pages should explicitly connect new AI/agentic concepts back to standard patterns.

**Frontmatter.** Populate `enterprise_analogs` with specific RFCs, grant types, or named patterns (e.g., `RFC 6749 §4.4 client_credentials`, `OIDC ID Token`, `RFC 8693 token exchange`, `SAML SP-initiated SSO`, `SCIM provisioning`). Use precise references, not generic terms like "OAuth." The field and the body sections share the same defaults: where the sections are required, the field is required.

**Body sections** (apply per the defaults below):

````markdown
## Relation to pre-AI IAM

How this maps to standard IAM patterns. Name the specific RFC, spec section, or grant type. Identify the practitioner mental model that carries over unchanged.

## Why pre-AI IAM is insufficient

The specific assumption or capability of the standard pattern that breaks down in AI/agentic contexts. State which property fails, why, and what the AI workflow needs that the standard pattern doesn't provide.
````

**Defaults by category:**
- `concepts/`, `topics/` — include both sections
- `incidents/` — include when a standard threat model has a clean analog (e.g., confused-deputy, CSRF, token replay)
- `references/`, `sources/` — include when the document's main contribution is bridging or identifying the gap
- `entities/` — usually skip

## `_index.md` format

Grouped by category in this order: concepts, topics, references, entities, incidents, sources. Within each group, alphabetical by slug. One line per page:

```
- [[<slug>]] — <one-line summary> · <status> · <confidence>
```

## `log.md` format

Append-only. Greppable via `grep '^## \[' log.md`.

```
## [YYYY-MM-DD] <ingest | query | lint> | <short title>
- <page slug>: <one-line summary of change>
- ...
```
