---
title: RFC 7591 — OAuth 2.0 Dynamic Client Registration Protocol
category: reference
status: stable
confidence: high
aliases: [RFC 7591, Dynamic Client Registration, DCR, /register]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [client-registration, oauth-client-id-metadata-documents, openid-connect-dynamic-client-registration, mcp-authorization]
tags: [oauth, client-registration, dcr, spec, ietf, reference]
---

# RFC 7591 — OAuth 2.0 Dynamic Client Registration Protocol

**RFC 7591** lets a client register with an authorization server at runtime by POSTing client metadata to a registration endpoint (`/register`) and receiving a `client_id` (and possibly a secret). It was the original answer to "how does a client get an ID without a human admin?"

## Role in MCP Authorization

**MAY** be supported, but **deprecated** in the MCP profile. Per the [[mcp-authorization-overview]], Dynamic Client Registration is "retained for backwards compatibility with authorization servers that do not support [[oauth-client-id-metadata-documents|Client ID Metadata Documents]]." It is one of the three [[client-registration]] mechanisms, the lowest-priority of the three. The OIDC profile of the same idea is [[openid-connect-dynamic-client-registration]].

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc7591>
