---
title: Token Theft
category: concept
status: stable
confidence: high
aliases: [token theft, stolen token, leaked token, token leakage, token exfiltration]
enterprise_analogs: [OAuth 2.1 §7.1 token storage, OAuth 2.1 §4.3.1 refresh token rotation, RFC 9700 OAuth 2.0 Security BCP, bearer token replay]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations]
related: [token-audience-binding, token-passthrough, mcp-authorization, security-considerations, oauth-2-1, machine-identity, public-vs-confidential-client]
tags: [security, token, threat-model, core-concept]
---

# Token Theft

**Token theft** is the threat that an attacker obtains an access or refresh token and replays it. Because OAuth bearer tokens are, by design, usable by whoever holds them, a stolen token lets an attacker make requests that "appear legitimate to resource servers" ([[mcp-authorization-security-considerations]], *Token Theft*). The exposure surfaces MCP calls out are tokens **stored by the client** and tokens **cached or logged on the server**.

## How MCP mitigates it

The mitigations are layered — reduce the chance of leakage, then reduce the value of a leaked token:

- **Secure storage (MUST).** Clients and servers MUST implement secure token storage and follow OAuth best practices ([[oauth-2-1|OAuth 2.1]] §7.1). Logging or caching raw tokens server-side is the named hazard.
- **Short-lived access tokens (SHOULD).** Authorization servers SHOULD issue short-lived access tokens so a leaked token expires quickly.
- **Refresh-token rotation for public clients (MUST).** For public clients — the typical MCP client — ASes MUST rotate refresh tokens ([[oauth-2-1|OAuth 2.1]] §4.3.1): each refresh returns a new refresh token and invalidates the old one, so a stolen refresh token is detectable (a replay of the retired token signals theft) and bounded in lifetime.

[[token-audience-binding|Audience binding]] is the complementary control: a stolen token still only works at the audience it was minted for, limiting where theft can be cashed in.

## Relation to pre-AI IAM

This is ordinary OAuth bearer-token hygiene. Token storage (OAuth 2.1 §7.1), short token lifetimes, and refresh-token rotation (OAuth 2.1 §4.3.1, also in the OAuth 2.0 Security BCP, RFC 9700) are the same controls a practitioner applies to any public OAuth client — a SPA or mobile app that cannot hold a client secret. The "bearer token = cash" mental model carries over unchanged: protect it in storage, keep it short-lived, and rotate the long-lived refresh credential.

## Why pre-AI IAM is insufficient

The controls themselves are unchanged; the **exposure surface widens** in agentic deployments. An MCP client brokers tokens for many independently operated servers and may persist them across sessions; an MCP server may inadvertently log tokens while tracing tool calls. Agents also process untrusted content that could exfiltrate a token if storage or logging is sloppy. MCP's response is to make storage discipline and refresh rotation **normative MUSTs** rather than recommendations, and to lean on mandatory [[token-audience-binding|audience binding]] so that even a stolen token cannot become a cross-server [[confused-deputy]] weapon. For the prohibition on *deliberately* relaying tokens, see [[token-passthrough]].
