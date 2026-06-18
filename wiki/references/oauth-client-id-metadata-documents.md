---
title: OAuth Client ID Metadata Documents (draft)
category: reference
status: evolving
confidence: high
aliases: [Client ID Metadata Documents, CIMD, URL client_id, draft-ietf-oauth-client-id-metadata-document]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [client-registration, rfc-7591-dynamic-client-registration, mcp-authorization, security-considerations]
tags: [oauth, client-registration, draft, ietf, reference]
---

# OAuth Client ID Metadata Documents (draft)

**Client ID Metadata Documents** (draft-ietf-oauth-client-id-metadata-document-00) let a client use an **HTTPS URL as its `client_id`**. The authorization server detects the URL-formatted ID, dereferences it to fetch a JSON metadata document (declaring `redirect_uris`, etc.), and validates that document — no prior registration call required. It is a registration-free alternative to [[rfc-7591-dynamic-client-registration|Dynamic Client Registration]].

## Role in MCP Authorization

The **preferred** client-identity mechanism. Per the [[mcp-authorization-overview]], authorization servers and MCP clients **SHOULD** support Client ID Metadata Documents, and the profile deprecates Dynamic Client Registration in their favor. One of the three [[client-registration]] mechanisms. Their security (validating the fetched document, the `redirect_uris`, and the URL itself) is called out as a dedicated topic in [[security-considerations]].

## Link

- Draft: <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00>

## Notes

Early IETF draft (`-00`); details are subject to change.
