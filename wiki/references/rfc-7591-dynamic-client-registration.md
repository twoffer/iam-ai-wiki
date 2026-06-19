---
title: RFC 7591 — OAuth 2.0 Dynamic Client Registration Protocol
category: reference
status: stable
confidence: high
aliases: [RFC 7591, Dynamic Client Registration, DCR, /register]
enterprise_analogs: []
last_updated: 2026-06-19
sources: [mcp-authorization-client-registration, mcp-authorization-overview]
related: [client-registration, oauth-client-id-metadata-documents, openid-connect-dynamic-client-registration, rfc-8414-authorization-server-metadata, mcp-authorization]
tags: [oauth, client-registration, dcr, spec, ietf, reference]
---

# RFC 7591 — OAuth 2.0 Dynamic Client Registration Protocol

**RFC 7591** lets a client register with an authorization server at runtime by POSTing client metadata to a registration endpoint (`/register`) and receiving a `client_id` (and possibly a secret). It was the original answer to "how does a client get an ID without a human admin?"

## Role in MCP Authorization

**MAY** be supported, but **deprecated** in the MCP profile. Per the [[mcp-authorization-overview]], Dynamic Client Registration is "retained for backwards compatibility with authorization servers that do not support [[oauth-client-id-metadata-documents|Client ID Metadata Documents]]." It is the lowest-priority of the three [[client-registration]] mechanisms, used as a fallback only when the AS exposes a `registration_endpoint` ([[rfc-8414-authorization-server-metadata|AS Metadata]]) and does not advertise CIMD support. The OIDC profile of the same idea is [[openid-connect-dynamic-client-registration]].

## Constraints under the MCP profile

The Client Registration document ([[mcp-authorization-client-registration]]) adds two operational rules for DCR clients:

- **`application_type`.** Where an AS supports OIDC, it may constrain redirect URIs by the [[openid-connect-dynamic-client-registration|OIDC]] `application_type`. Clients **MUST** set it (`native` for desktop/mobile/CLI/`localhost`; `web` for remote browser apps), because omitting it defaults to `web` under OIDC and can reject native-style redirect URIs. Clients **MUST** handle such rejections and **MAY** retry with a corrected `application_type` or redirect URIs.
- **Authorization Server Binding.** Persisted DCR credentials **MUST** be keyed to the issuing AS `issuer`; when a server's AS changes, the client **MUST NOT** reuse them and **MUST** re-register. See [[client-registration]].

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc7591>
