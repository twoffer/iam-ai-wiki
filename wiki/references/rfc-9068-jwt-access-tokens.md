---
title: RFC 9068 — JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens
category: reference
status: stable
confidence: high
aliases: ["RFC 9068", "JWT access token profile", "JWT access tokens", "aud claim"]
enterprise_analogs: []
last_updated: 2026-06-19
sources: ["mcp-authorization-security-considerations"]
related: ["token-audience-binding", "token-passthrough", "rfc-8707-resource-indicators", "security-considerations", "mcp-authorization"]
tags: ["oauth", "jwt", "audience", "token", "spec", "ietf", "reference"]
---

# RFC 9068 — JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens

**RFC 9068** standardizes the structure of an OAuth 2.0 access token expressed as a JWT, including the claims a resource server validates — notably the **`aud` (audience) claim** that names the resource(s) the token is intended for, along with `iss`, `exp`, `sub`, `client_id`, and `scope`. It gives resource servers a well-defined way to check, from the token itself, that they are the intended recipient.

## Role in MCP Authorization

Cited by the [[mcp-authorization-security-considerations|Security Considerations]] document (*Access Token Privilege Restriction*) as the basis for **audience validation**: an MCP server that fails to verify the audience claim "may accept tokens originally issued for other services," breaking an OAuth security boundary. It underpins the [[token-audience-binding]] requirement on the server side — the `aud` claim is what a server checks to enforce that a token was minted for it — and the prohibition on [[token-passthrough]]. The client-side counterpart is the [[rfc-8707-resource-indicators|RFC 8707]] `resource` parameter that causes the AS to stamp that audience in the first place.

Note that MCP does not mandate JWT access tokens specifically; RFC 9068 is referenced as the canonical example of the audience claim. Servers validate tokens per [[oauth-2-1|OAuth 2.1]] §5.2, by introspection or local JWT validation, and either way MUST confirm they are the intended audience.

## Link

- RFC: <https://www.rfc-editor.org/rfc/rfc9068.html>
