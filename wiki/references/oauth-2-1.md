---
title: OAuth 2.1 (draft-ietf-oauth-v2-1)
category: reference
status: evolving
confidence: high
aliases: [OAuth 2.1, oauth-v2-1, draft-ietf-oauth-v2-1-13]
enterprise_analogs: [RFC 6749 OAuth 2.0, RFC 6750, RFC 7636 PKCE, RFC 8252]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations, mcp-authorization-overview]
related: [mcp-authorization, proof-key-for-code-exchange, rfc-6750-bearer-token-usage, token-audience-binding, rfc-7636-pkce, security-considerations, token-theft, open-redirection, ietf-oauth-working-group]
tags: [oauth, spec, ietf, reference]
---

# OAuth 2.1 (draft-ietf-oauth-v2-1)

**OAuth 2.1** is an [[ietf-oauth-working-group|IETF OAuth Working Group]] draft that consolidates OAuth 2.0 (RFC 6749) and its widely adopted extensions and security best practices into a single, simplified document. Key consolidations: **PKCE is mandatory** for all authorization-code clients, the **implicit** and **resource-owner password** grants are removed, exact redirect-URI matching is required, and bearer-token usage follows RFC 6750.

Referenced as `draft-ietf-oauth-v2-1-13` in the [[mcp-authorization-overview]].

## Role in MCP Authorization

It is the **baseline**: MCP authorization servers MUST implement OAuth 2.1 for both confidential and public clients, and MCP servers act as OAuth 2.1 **resource servers**. Token usage and validation in MCP conform to OAuth 2.1 §5 (Resource Requests), §5.2 (token validation), and §5.3 (error handling). See [[mcp-authorization]] and [[proof-key-for-code-exchange]].

## Security sections MCP requires

The [[mcp-authorization-security-considerations|Security Considerations]] document requires implementers to follow OAuth 2.1 **§7 "Security Considerations"** in full, and pins specific sub-sections to MCP MUSTs:

| OAuth 2.1 § | MCP use |
| --- | --- |
| §1.5 Communication Security | HTTPS on all AS endpoints; redirect URIs `localhost` or HTTPS |
| §4.1.1 | PKCE `S256` challenge method ([[proof-key-for-code-exchange]]) |
| §4.3.1 Token Endpoint Extension | refresh-token rotation for public clients ([[token-theft]]) |
| §5.2 | inbound access-token validation ([[token-audience-binding]]) |
| §7.1 | secure token storage ([[token-theft]]) |
| §7.5 / §7.5.2 | authorization-code protection via PKCE |
| §7.12.2 | open-redirection precautions ([[open-redirection]]) |

## Link

- Draft: <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13>

## Notes

A draft (evolving); section numbering and normative levels can shift between revisions. The overview cites both `-13` and, for one refresh-token clause, `-14`.
