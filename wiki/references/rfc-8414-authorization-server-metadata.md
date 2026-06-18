---
title: RFC 8414 — OAuth 2.0 Authorization Server Metadata
category: reference
status: stable
confidence: high
aliases: [RFC 8414, Authorization Server Metadata, AS metadata, oauth-authorization-server]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [authorization-server-discovery, openid-connect-discovery, mcp-authorization, rfc-9207-authorization-server-issuer-identification]
tags: [oauth, discovery, metadata, spec, ietf, reference]
---

# RFC 8414 — OAuth 2.0 Authorization Server Metadata

**RFC 8414** defines a JSON metadata document — at `.well-known/oauth-authorization-server` — by which an authorization server advertises its endpoints (authorization, token, registration), supported grant types, scopes, and capabilities (including `authorization_response_iss_parameter_supported`, the [[rfc-9207-authorization-server-issuer-identification|RFC 9207]] flag).

## Role in MCP Authorization

One of the two AS-discovery mechanisms. Per the [[mcp-authorization-overview]], an MCP authorization server MUST provide **at least one** of RFC 8414 or [[openid-connect-discovery|OpenID Connect Discovery]], and MCP clients MUST support **both**, trying the OAuth 2.0 endpoint before the OIDC one. The client records the `issuer` from this document for later `iss` validation. See [[authorization-server-discovery]].

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc8414>
