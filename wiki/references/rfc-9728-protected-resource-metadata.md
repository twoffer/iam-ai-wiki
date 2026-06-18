---
title: RFC 9728 — OAuth 2.0 Protected Resource Metadata
category: reference
status: stable
confidence: high
aliases: [RFC 9728, Protected Resource Metadata, PRM, oauth-protected-resource, resource_metadata]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [authorization-server-discovery, mcp-authorization, scope-selection-strategy, rfc-8414-authorization-server-metadata, token-audience-binding]
tags: [oauth, discovery, metadata, spec, ietf, reference]
---

# RFC 9728 — OAuth 2.0 Protected Resource Metadata

**RFC 9728** lets a protected resource (resource server) publish a metadata document — at `.well-known/oauth-protected-resource` — describing itself: which authorization server(s) it trusts, its resource identifier, supported scopes (`scopes_supported`), and more. A `WWW-Authenticate` challenge points clients to it via the `resource_metadata` parameter.

## Role in MCP Authorization

**Mandatory on every MCP server.** Per the [[mcp-authorization-overview]], MCP servers MUST implement RFC 9728 and clients MUST use it for [[authorization-server-discovery]]. It is the first hop of discovery: the `401` response carries `resource_metadata`, the client fetches the document, and extracts the authorization server(s). Its `scopes_supported` feeds the [[scope-selection-strategy]], and its resource identifier aligns with the [[canonical-server-uri|canonical URI]] / [[token-audience-binding|audience]].

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc9728>
