---
title: Token Passthrough (Anti-Pattern)
category: concept
status: stub
confidence: high
aliases: [token passthrough, token pass-through, token forwarding, confused token relay]
enterprise_analogs: [RFC 8707 audience restriction, OAuth 2.1 §5.2 audience validation, RFC 8693 token exchange (the correct alternative)]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [token-audience-binding, confused-deputy, mcp-authorization, security-considerations]
tags: [security, token, anti-pattern, stub]
---

# Token Passthrough (Anti-Pattern)

**Token passthrough** is the anti-pattern in which a server accepts an access token that was not issued for it, or forwards a token it received onward to another service. It breaks [[token-audience-binding|audience binding]] and turns intermediaries into [[confused-deputy|confused deputies]]: a token's audience no longer constrains where the user's authority can be exercised.

## How MCP prohibits it

The [[mcp-authorization-overview]] (*Access Token Usage*) states the rule plainly:

- MCP clients **MUST NOT** send a server any token other than one issued by that server's own authorization server.
- MCP servers **MUST** only accept tokens valid for their own resources, and **MUST NOT accept or transit any other tokens.**

The correct way to cross a trust boundary is not to pass a token through, but to obtain a *new*, properly audience-bound token for the next hop (e.g., [[token-audience-binding|RFC 8693 token exchange]]).

## Relation to pre-AI IAM

The prohibition is a direct consequence of RFC 8707 audience restriction and OAuth 2.1 §5.2 audience validation. Enterprises learned the same lesson in API-gateway and microservice designs: blindly forwarding a caller's token downstream ("token relay") creates confused deputies and over-broad authority. Token exchange exists precisely to replace passthrough.

## Why pre-AI IAM is insufficient

Token passthrough is tempting in agentic systems because an agent naturally relays between many servers; an unrestricted bearer token "just works" everywhere, which is exactly the hazard. With agents steerable by untrusted input and brokering dynamically discovered servers, passthrough converts a single stolen or misdirected token into cross-service compromise. MCP therefore makes the prohibition normative rather than advisory. Fuller treatment is expected from [[security-considerations]].

> **Status:** stub. Seeded from the MCP overview's "MUST NOT accept or transit" rule; to be expanded when the Security Considerations document is ingested.
