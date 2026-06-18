---
title: Scope Selection Strategy
category: topic
status: stable
confidence: high
aliases: [scope selection, scope strategy, scopes_supported, scope challenge handling]
enterprise_analogs: [OAuth scopes (RFC 6749 §3.3), RFC 6750 §3 WWW-Authenticate scope, least privilege, RFC 9728 scopes_supported]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, step-up-authorization, human-in-the-loop-authorization, tool-use-authorization, rfc-6750-bearer-token-usage, rfc-9728-protected-resource-metadata]
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

## Relation to pre-AI IAM

Scopes are ordinary OAuth (RFC 6749 §3.3); advertising them via `WWW-Authenticate` is RFC 6750 §3; "request only what you need" is least privilege. A practitioner already designs APIs with granular scopes and challenges under-scoped callers.

## Why pre-AI IAM is insufficient

The defining difference: a traditional client is **built by a developer who knows the exact scopes** and hard-codes them; an MCP client is **general-purpose and domain-blind**, connecting to servers it has never seen. It therefore cannot pre-select scopes correctly and must lean on server-advertised challenges and runtime escalation. The strategy exists to let a non-domain-aware agent still honor least privilege, pushing the informed decision to the server's challenge and the user's consent rather than to client code.
