---
title: Token Audience Binding
category: concept
status: stable
confidence: high
aliases: [audience restriction, audience validation, resource indicators, token audience]
enterprise_analogs: [RFC 8707 Resource Indicators, JWT `aud` claim (RFC 7519), OAuth 2.1 §5.2 token validation, RFC 9068 JWT access tokens]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, canonical-server-uri, rfc-8707-resource-indicators, token-passthrough, confused-deputy, authorization-server-mix-up, delegated-authorization]
tags: [oauth, audience, token, security, core-concept]
---

# Token Audience Binding

**Token audience binding** is the practice of issuing an access token *for a specific resource* and having that resource accept the token *only if it is the intended audience*. It is the mechanism that stops a token obtained for one server from being replayed against another. In MCP it is **mandatory on both sides** of the exchange ([[mcp-authorization-overview]]).

## How MCP requires it

- **Client side (request binding).** MCP clients MUST implement [[rfc-8707-resource-indicators|Resource Indicators for OAuth 2.0 (RFC 8707)]]: the `resource` parameter naming the target server's [[canonical-server-uri|canonical URI]] MUST be sent in both the authorization request and the token request, regardless of whether the AS supports it (*Resource Parameter Implementation*).
- **Server side (audience validation).** MCP servers, acting as OAuth 2.1 resource servers, MUST validate that an access token "was issued specifically for them as the intended audience" per RFC 8707 §2, and MUST reject anything else with `401`. Servers MUST only accept tokens valid for their own resources and MUST NOT accept or transit any other tokens — the [[token-passthrough]] prohibition (*Access Token Usage*).

Together these make the token a key cut for one lock: the AS stamps the audience from the `resource` parameter; the resource server checks the stamp.

## Why it matters

Audience binding is the primary structural defense against the [[confused-deputy]] problem in a multi-server agent: even if a malicious or compromised server (or injected instruction) induces the client to send a token somewhere unintended, an audience-bound token is useless at the wrong server. It also limits blast radius from [[authorization-server-mix-up]] and token theft.

## Relation to pre-AI IAM

This is exactly **RFC 8707 Resource Indicators** plus standard resource-server audience checking ([[oauth-2-1]] §5.2; the JWT `aud` claim, RFC 7519 / RFC 9068). Enterprises already use `resource`/`audience` parameters to keep a token minted for the billing API from working against the HR API. The practitioner model — "a token is scoped to an audience, and each API rejects tokens not minted for it" — transfers directly.

## Why pre-AI IAM is insufficient

In classic deployments audience restriction is **optional and frequently skipped**: a single bearer token is often accepted by several first-party APIs behind one gateway, because all parties trust each other and were integrated by hand. In the agentic model the client brokers tokens for *many independently operated* servers it discovered at runtime, while being steerable by untrusted content. A reused, unrestricted bearer token then becomes a confused-deputy weapon. MCP's response is to make what was optional **mandatory**: every token MUST be audience-bound and every server MUST validate audience, so that no single token grants cross-server authority. See [[mcp-authorization|Why pre-AI IAM is insufficient]].
