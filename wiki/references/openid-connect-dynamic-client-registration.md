---
title: OpenID Connect Dynamic Client Registration 1.0
category: reference
status: stable
confidence: high
aliases: [OIDC Dynamic Client Registration, OpenID Connect Registration, OIDC DCR]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [client-registration, rfc-7591-dynamic-client-registration, oauth-client-id-metadata-documents, openid-connect-discovery, mcp-authorization]
tags: [oidc, client-registration, dcr, spec, openid-foundation, reference]
---

# OpenID Connect Dynamic Client Registration 1.0

**OpenID Connect Dynamic Client Registration 1.0** (OpenID Foundation) is the OIDC profile of runtime client registration — the OIDC counterpart to [[rfc-7591-dynamic-client-registration|RFC 7591]]. A client registers by submitting metadata to the OP and receives client credentials.

## Role in MCP Authorization

Listed among the standards the MCP profile builds on ([[mcp-authorization-overview]], *Standards Compliance*). Like RFC 7591, it belongs to the **Dynamic Client Registration** path of [[client-registration]], which the profile **deprecates** in favor of [[oauth-client-id-metadata-documents|Client ID Metadata Documents]], retaining it for backward compatibility.

## Link

- Spec: <https://openid.net/specs/openid-connect-registration-1_0.html>
