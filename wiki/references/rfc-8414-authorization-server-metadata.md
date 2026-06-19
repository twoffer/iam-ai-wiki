---
title: RFC 8414 — OAuth 2.0 Authorization Server Metadata
category: reference
status: stable
confidence: high
aliases: [RFC 8414, Authorization Server Metadata, AS metadata, oauth-authorization-server]
enterprise_analogs: []
last_updated: 2026-06-19
sources: [mcp-authorization-server-discovery, mcp-authorization-overview]
related: [authorization-server-discovery, openid-connect-discovery, mcp-authorization, rfc-9207-authorization-server-issuer-identification, authorization-server-mix-up]
tags: [oauth, discovery, metadata, spec, ietf, reference]
---

# RFC 8414 — OAuth 2.0 Authorization Server Metadata

**RFC 8414** defines a JSON metadata document — at `.well-known/oauth-authorization-server` — by which an authorization server advertises its endpoints (authorization, token, registration), supported grant types, scopes, and capabilities (including `authorization_response_iss_parameter_supported`, the [[rfc-9207-authorization-server-issuer-identification|RFC 9207]] flag).

## Role in MCP Authorization

One of the two AS-discovery mechanisms. Per the [[mcp-authorization-overview]], an MCP authorization server MUST provide **at least one** of RFC 8414 or [[openid-connect-discovery|OpenID Connect Discovery]], and MCP clients MUST support **both**, trying the OAuth 2.0 endpoint before the OIDC one. The client records the `issuer` from this document for later `iss` validation. See [[authorization-server-discovery]].

The [[mcp-authorization-server-discovery|AS Discovery]] document fixes the construction and validation rules:

- **Default suffix only.** MCP uses the default `oauth-authorization-server` well-known suffix from [§3.1](https://datatracker.ietf.org/doc/html/rfc8414#section-3.1) and defines no MCP-specific suffix. For an issuer with a path component (`https://auth.example.com/tenant1`), the OAuth 2.0 endpoint uses **path insertion**: `https://auth.example.com/.well-known/oauth-authorization-server/tenant1`. The [§5 compatibility notes](https://datatracker.ietf.org/doc/html/rfc8414#section-5) govern the OIDC fallbacks the client also tries.
- **Issuer validation ([§3.3](https://datatracker.ietf.org/doc/html/rfc8414#section-3.3)).** The `issuer` value inside the fetched document MUST be identical to the issuer identifier used to build the well-known URL; otherwise the client MUST reject the document. This discovery-layer check is distinct from RFC 9207 authorization-response `iss` validation — see [[authorization-server-mix-up]].

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc8414>
