---
title: Scope Selection Strategy
category: topic
status: stable
confidence: high
aliases: [scope selection, scope strategy, scope minimization, scopes_supported, scope challenge handling]
enterprise_analogs: [OAuth scopes (RFC 6749 §3.3), RFC 6750 §3 WWW-Authenticate scope, least privilege, RFC 9728 scopes_supported, blast-radius reduction]
last_updated: 2026-07-08
sources: [mcp-authorization-overview, mcp-security-best-practices]
related: [mcp-authorization, step-up-authorization, human-in-the-loop-authorization, tool-use-authorization, rfc-6750-bearer-token-usage, rfc-9728-protected-resource-metadata, token-theft, mcp-security-best-practices]
tags: [oauth, scopes, least-privilege, mcp]
---

# Scope Selection Strategy

The **scope selection strategy** is how an [[mcp-authorization|MCP client]] decides *which* OAuth scopes to request, given that a general-purpose client lacks domain knowledge to choose scopes intelligently. The strategy follows the principle of least privilege while minimizing user friction ([[mcp-authorization-overview]], *Scope Selection Strategy*).

## Server-side: advertising required scopes

MCP servers SHOULD include a `scope` parameter in the `WWW-Authenticate` header ([[rfc-6750-bearer-token-usage|RFC 6750]] §3) to tell the client exactly which scopes the requested resource needs. Example:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer resource_metadata="https://mcp.example.com/.well-known/oauth-protected-resource",
                         scope="files:read"
```

The challenged `scope` set MAY match `scopes_supported`, be a subset, a superset, or an unrelated set. Clients **MUST NOT** assume any set relationship and **MUST** treat the challenged scopes as authoritative for the current operation. Servers SHOULD be consistent but need not surface every dynamically issued scope through `scopes_supported`. `scopes_supported` is meant to be the **minimal set for basic functionality**, with more requested incrementally via [[step-up-authorization]].

## Client-side: priority order

During the initial handshake the client SHOULD request only what it needs, in this priority order:

1. **Use the `scope`** from the initial `WWW-Authenticate` 401, if present.
2. **Otherwise**, request all scopes in `scopes_supported` from the Protected Resource Metadata; omit `scope` entirely if `scopes_supported` is undefined.

Requesting all advertised scopes (when no specific challenge is given) defers the actual permission decision to the AS and the end-user at the consent screen ([[human-in-the-loop-authorization]]) — appropriate because the client cannot make a domain-specific judgment itself.

## Runtime: insufficient-scope challenges

When a client already holds a token but hits an operation it is under-scoped for, the server SHOULD return `403` with `error="insufficient_scope"` and a `scope` challenge. The client then performs [[step-up-authorization]], unioning old and new scopes. Servers SHOULD emit all scopes for a single operation at once (not drip them one per round-trip) and SHOULD be consistent in their inclusion strategy (minimum / recommended / extended). Scope accumulation across operations is the client's responsibility, keeping servers stateless about client scope sets.

## Scope minimization: the security case

The [[mcp-security-best-practices|Security Best Practices]] guide (*Scope Minimization*) supplies the threat model behind the strategy. The attack framing: a token carrying broad scopes (`files:*`, `db:*`, `admin:*`) — granted up front because the server exposed everything in `scopes_supported` and the client requested it all — is obtained via log leakage, memory scraping, or local interception. The consequences of poor scope design:

- **Expanded blast radius** — the stolen token reaches unrelated tools and resources, with immediate **privilege chaining** and no further elevation prompts.
- **Revocation friction** — revoking a max-privilege token disrupts every workflow at once.
- **Audit noise** — one omnibus scope masks per-operation user intent, and without elevation metrics, over-broad requests become normalized ("scope inflation blindness").
- **Consent abandonment** — users decline dialogs listing excessive scopes ([[human-in-the-loop-authorization]]).

The prescribed model is progressive least privilege: a **minimal initial scope set** (e.g., `mcp:tools-basic` for low-risk discovery/read operations), **incremental elevation** via targeted `WWW-Authenticate` `scope="..."` challenges when privileged operations are first attempted ([[step-up-authorization]]), and **down-scoping tolerance** — servers accept reduced-scope tokens, and the AS MAY issue a subset of what was requested. Servers should emit precise challenges rather than the full catalog and log elevation events (scope requested, subset granted) with correlation IDs; clients should begin with baseline scopes and cache recent denials to avoid repeated elevation loops.

The guide's *Common Mistakes* list is a useful design lint: publishing all possible scopes in `scopes_supported` (reinforcing that it should stay the minimal-basic set); wildcard or omnibus scopes (`*`, `all`, `full-access`); bundling unrelated privileges to preempt future prompts; returning the entire scope catalog in every challenge; silently changing scope semantics without versioning; and treating claimed scopes in a token as sufficient **without server-side authorization logic** — scopes gate entry, but the resource server still authorizes each operation (see [[tool-use-authorization]]).

## Relation to pre-AI IAM

Scopes are ordinary OAuth (RFC 6749 §3.3); advertising them via `WWW-Authenticate` is RFC 6750 §3; "request only what you need" is least privilege. A practitioner already designs APIs with granular scopes and challenges under-scoped callers.

## Why pre-AI IAM is insufficient

The defining difference: a traditional client is **built by a developer who knows the exact scopes** and hard-codes them; an MCP client is **general-purpose and domain-blind**, connecting to servers it has never seen. It therefore cannot pre-select scopes correctly and must lean on server-advertised challenges and runtime escalation. The strategy exists to let a non-domain-aware agent still honor least privilege, pushing the informed decision to the server's challenge and the user's consent rather than to client code.
