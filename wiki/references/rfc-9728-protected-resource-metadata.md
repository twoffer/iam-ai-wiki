---
title: RFC 9728 — OAuth 2.0 Protected Resource Metadata
category: reference
status: stable
confidence: high
aliases: [RFC 9728, Protected Resource Metadata, PRM, oauth-protected-resource, resource_metadata]
enterprise_analogs: []
last_updated: 2026-06-19
sources: [mcp-authorization-server-discovery, mcp-authorization-overview]
related: [authorization-server-discovery, mcp-authorization, scope-selection-strategy, rfc-8414-authorization-server-metadata, token-audience-binding, client-registration]
tags: [oauth, discovery, metadata, spec, ietf, reference]
---

# RFC 9728 — OAuth 2.0 Protected Resource Metadata

**RFC 9728** lets a protected resource (resource server) publish a metadata document — at `.well-known/oauth-protected-resource` — describing itself: which authorization server(s) it trusts (the `authorization_servers` field), its resource identifier, supported scopes (`scopes_supported`), and more. A `WWW-Authenticate` challenge points clients to it via the `resource_metadata` parameter.

## Role in MCP Authorization

**Mandatory on every MCP server.** Per the [[mcp-authorization-overview]], MCP servers MUST implement RFC 9728 and clients MUST use it for [[authorization-server-discovery]]. It is the first hop of discovery: the metadata document MUST include an `authorization_servers` field naming at least one AS, the client fetches the document, and extracts the authorization server(s). Its `scopes_supported` feeds the [[scope-selection-strategy]], and its resource identifier aligns with the [[canonical-server-uri|canonical URI]] / [[token-audience-binding|audience]].

The [[mcp-authorization-server-discovery|AS Discovery]] document adds the operational detail:

- **Two location mechanisms** (server implements one; client supports both): the `resource_metadata` parameter in the `WWW-Authenticate` header on a `401` (§5.1), or the well-known URI served either at the MCP endpoint's path (`https://example.com/public/mcp` → `https://example.com/.well-known/oauth-protected-resource/public/mcp`) or at the root (`https://example.com/.well-known/oauth-protected-resource`). Clients prefer the header, then fall back to the well-known URIs (path-based, then root).
- **Multiple authorization servers.** A document may list several, each an independent AS; the client selects which to use per [§7.6](https://datatracker.ietf.org/doc/html/rfc9728#name-authorization-servers) and keeps registration state isolated per AS (see [[client-registration]]).

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc9728>
