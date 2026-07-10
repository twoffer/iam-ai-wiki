---
title: RFC 8707 — Resource Indicators for OAuth 2.0
category: reference
status: stable
confidence: high
aliases: ["RFC 8707", "Resource Indicators", "resource parameter", "audience parameter"]
enterprise_analogs: []
last_updated: 2026-06-19
sources: ["mcp-authorization-security-considerations", "mcp-authorization-overview"]
related: ["token-audience-binding", "canonical-server-uri", "mcp-authorization", "rfc-9728-protected-resource-metadata", "rfc-9068-jwt-access-tokens", "token-passthrough"]
tags: ["oauth", "resource-indicators", "audience", "spec", "ietf", "reference"]
---

# RFC 8707 — Resource Indicators for OAuth 2.0

**RFC 8707** defines the `resource` request parameter, by which a client tells the authorization server the specific protected resource(s) a token is intended for. The AS uses it to mint an **audience-restricted** token, so the token cannot be used at a resource it was not requested for.

## Role in MCP Authorization

**Mandatory** in MCP and central to [[token-audience-binding]]. Per the [[mcp-authorization-overview]] (*Resource Parameter Implementation*), clients MUST include `resource` in **both** the authorization and token requests, identifying the target server by its [[canonical-server-uri|canonical URI]] (RFC 8707 §2), and MUST send it **regardless of AS support**. MCP servers MUST validate that a token's audience is themselves (RFC 8707 §2). The canonical URI aligns with the `resource` identifier of [[rfc-9728-protected-resource-metadata|RFC 9728]] (the [[mcp-authorization-security-considerations|Security Considerations]] document pins this to [RFC 9728 §7.4](https://datatracker.ietf.org/doc/html/rfc9728#section-7.4)).

The Security Considerations document frames `resource` as the client-side half of *Access Token Privilege Restriction*: the parameter causes the AS to stamp the audience, and the server checks that stamp ([[rfc-9068-jwt-access-tokens|RFC 9068]] `aud`) — without it, audience validation has nothing to validate and [[token-passthrough]] becomes possible.

## Link

- RFC: <https://www.rfc-editor.org/rfc/rfc8707.html>
