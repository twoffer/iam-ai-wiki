---
title: OAuth 2.1 (draft-ietf-oauth-v2-1)
category: reference
status: evolving
confidence: high
aliases: [OAuth 2.1, oauth-v2-1, draft-ietf-oauth-v2-1-13]
enterprise_analogs: [RFC 6749 OAuth 2.0, RFC 6750, RFC 7636 PKCE, RFC 8252]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, proof-key-for-code-exchange, rfc-6750-bearer-token-usage, token-audience-binding, rfc-7636-pkce, ietf-oauth-working-group]
tags: [oauth, spec, ietf, reference]
---

# OAuth 2.1 (draft-ietf-oauth-v2-1)

**OAuth 2.1** is an [[ietf-oauth-working-group|IETF OAuth Working Group]] draft that consolidates OAuth 2.0 (RFC 6749) and its widely adopted extensions and security best practices into a single, simplified document. Key consolidations: **PKCE is mandatory** for all authorization-code clients, the **implicit** and **resource-owner password** grants are removed, exact redirect-URI matching is required, and bearer-token usage follows RFC 6750.

Referenced as `draft-ietf-oauth-v2-1-13` in the [[mcp-authorization-overview]].

## Role in MCP Authorization

It is the **baseline**: MCP authorization servers MUST implement OAuth 2.1 for both confidential and public clients, and MCP servers act as OAuth 2.1 **resource servers**. Token usage and validation in MCP conform to OAuth 2.1 §5 (Resource Requests), §5.2 (token validation), and §5.3 (error handling). See [[mcp-authorization]] and [[proof-key-for-code-exchange]].

## Link

- Draft: <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13>

## Notes

A draft (evolving); section numbering and normative levels can shift between revisions. The overview cites both `-13` and, for one refresh-token clause, `-14`.
