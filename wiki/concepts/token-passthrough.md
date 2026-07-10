---
title: Token Passthrough (Anti-Pattern)
category: concept
status: stable
confidence: high
aliases: ["token passthrough", "token pass-through", "token forwarding", "confused token relay"]
enterprise_analogs: ["RFC 8707 audience restriction", "OAuth 2.1 §5.2 audience validation", "RFC 9068 JWT audience claim", "RFC 8693 token exchange (the correct alternative)"]
last_updated: 2026-07-08
sources: ["mcp-security-best-practices", "mcp-authorization-security-considerations", "mcp-authorization-overview"]
related: ["token-audience-binding", "confused-deputy", "mcp-authorization", "security-considerations", "rfc-9068-jwt-access-tokens", "mcp-security-best-practices"]
tags: ["security", "token", "anti-pattern", "core-concept"]
---

# Token Passthrough (Anti-Pattern)

**Token passthrough** is the anti-pattern in which a server accepts an access token that was not issued for it, or forwards a token it received onward to another service. It breaks [[token-audience-binding|audience binding]] and turns intermediaries into [[confused-deputy|confused deputies]]: a token's audience no longer constrains where the user's authority can be exercised.

## How MCP prohibits it

The [[mcp-authorization-overview]] (*Access Token Usage*) states the rule plainly:

- MCP clients **MUST NOT** send a server any token other than one issued by that server's own authorization server.
- MCP servers **MUST** only accept tokens valid for their own resources, and **MUST NOT accept or transit any other tokens.**

The correct way to cross a trust boundary is not to pass a token through, but to obtain a *new*, properly audience-bound token for the next hop (e.g., [[token-audience-binding|RFC 8693 token exchange]]).

## The two failure modes (Security Considerations)

The [[mcp-authorization-security-considerations|Security Considerations]] document (*Access Token Privilege Restriction*) decomposes the anti-pattern into two distinct failures:

1. **Audience validation failures.** A server that does not verify the audience claim ([[rfc-9068-jwt-access-tokens|RFC 9068]]) accepts tokens issued for *other* services. This alone breaks an OAuth security boundary, even before any forwarding.
2. **Token passthrough proper.** A server that *also* forwards those unmodified tokens to downstream services turns itself into a [[confused-deputy]]: the downstream API may trust the token "as if it came from the MCP server" or assume the upstream already validated it.

## The upstream-API rule

The sharpest normative statement concerns an MCP server that itself calls upstream APIs:

> If the MCP server makes requests to upstream APIs, it may act as an OAuth **client** to them. The access token used at the upstream API is a **separate** token, issued by the upstream authorization server. The MCP server **MUST NOT** pass through the token it received from the MCP client.

In other words, the server wears two hats — resource server to its MCP client, OAuth client to the upstream — and each hat carries its own audience-bound token. This is the operational form of "obtain a new token for the next hop" and the concrete defense against the proxy/intermediary [[confused-deputy]] case.

## Why it is forbidden — the four risk classes

The [[mcp-security-best-practices|Security Best Practices]] guide (to which the spec defers for the full rationale) decomposes the harm into four classes (*Token Passthrough → Risks*):

1. **Security-control circumvention.** The MCP server or downstream API may enforce rate limiting, request validation, or traffic monitoring keyed to the token's audience or other credential constraints; clients presenting upstream-issued tokens directly bypass all of it.
2. **Accountability and audit-trail failures.** With opaque upstream tokens the MCP server cannot identify or distinguish its own clients; the downstream resource server's logs attribute requests to the wrong source and identity; and a server that relays without validating claims (roles, privileges, audience) lets a stolen token turn it into a **data-exfiltration proxy**. All three make incident investigation and controls harder.
3. **Trust-boundary breakage.** The downstream server extends trust — assumptions about origin and client behavior — to specific entities. A token accepted by multiple services without proper validation means an attacker who compromises one service can pivot into the others on the same token.
4. **Future-compatibility risk.** Even a deliberate "pure proxy" will eventually need its own security controls; starting with proper token-audience separation is what keeps the security model able to evolve.

The guide's bottom line restates the spec: MCP servers **MUST NOT** accept any token that was not explicitly issued for them.

## Relation to pre-AI IAM

The prohibition is a direct consequence of RFC 8707 audience restriction and OAuth 2.1 §5.2 audience validation. Enterprises learned the same lesson in API-gateway and microservice designs: blindly forwarding a caller's token downstream ("token relay") creates confused deputies and over-broad authority. Token exchange exists precisely to replace passthrough.

## Why pre-AI IAM is insufficient

Token passthrough is tempting in agentic systems because an agent naturally relays between many servers; an unrestricted bearer token "just works" everywhere, which is exactly the hazard. With agents steerable by untrusted input and brokering dynamically discovered servers, passthrough converts a single stolen or misdirected token into cross-service compromise. MCP therefore makes the prohibition normative rather than advisory, and spells out the two-hats rule for servers that front upstream APIs. See [[security-considerations]] for the full normative context.
