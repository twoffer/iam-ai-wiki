---
title: OAuth Client ID Metadata Documents (draft)
category: reference
status: evolving
confidence: high
aliases: ["Client ID Metadata Documents", "CIMD", "URL client_id", "draft-ietf-oauth-client-id-metadata-document"]
enterprise_analogs: []
last_updated: 2026-06-19
sources: ["mcp-authorization-security-considerations", "mcp-authorization-client-registration", "mcp-authorization-overview"]
related: ["client-registration", "rfc-7591-dynamic-client-registration", "rfc-8414-authorization-server-metadata", "mcp-authorization", "security-considerations", "server-side-request-forgery", "open-redirection", "public-vs-confidential-client"]
tags: ["oauth", "client-registration", "cimd", "draft", "ietf", "reference"]
---

# OAuth Client ID Metadata Documents (draft)

**Client ID Metadata Documents** (draft-ietf-oauth-client-id-metadata-document-00) let a client use an **HTTPS URL as its `client_id`**. The authorization server detects the URL-formatted ID, dereferences it to fetch a JSON metadata document (declaring `redirect_uris`, etc.), and validates that document — no prior registration call required. It is a registration-free alternative to [[rfc-7591-dynamic-client-registration|Dynamic Client Registration]].

## Role in MCP Authorization

The **preferred** client-identity mechanism and the top capability-gated choice in the [[client-registration]] selection priority. Per the [[mcp-authorization-overview]], authorization servers and MCP clients **SHOULD** support Client ID Metadata Documents, and the profile deprecates Dynamic Client Registration in their favor. Their security (validating the fetched document, the `redirect_uris`, and the URL itself) is called out as a dedicated topic in [[security-considerations]].

## Requirements under the MCP profile

The Client Registration document ([[mcp-authorization-client-registration]]) restates the draft's key requirements:

- The `client_id` URL **MUST** use the `https` scheme and **contain a path component** (e.g., `https://example.com/client.json`).
- The metadata document **MUST** include at least `client_id`, `client_name`, and `redirect_uris`, and its `client_id` **MUST** match the document URL exactly.
- The client **MAY** authenticate with `private_key_jwt` (with JWKS) per draft §6.2; otherwise it is a public client (`token_endpoint_auth_method: "none"`).
- The authorization server **SHOULD** fetch the document for URL-formatted `client_id`s, **MUST** validate that its `client_id` matches the URL, **MUST** validate the request `redirect_uri` against the document's `redirect_uris`, **MUST** confirm the document is valid JSON with the required fields, and **SHOULD** cache it respecting HTTP cache headers.

**Discovery and portability.** An AS advertises support with `"client_id_metadata_document_supported": true` in its [[rfc-8414-authorization-server-metadata|Authorization Server Metadata]]. Unlike pre-registered or DCR credentials, CIMD client IDs are **portable across authorization servers** — they are self-hosted URLs resolved on demand, so no re-registration is needed when a server's AS changes (see *Authorization Server Binding* in [[client-registration]]).

## Security considerations (draft §6)

The [[mcp-authorization-security-considerations|Security Considerations]] document (*Client ID Metadata Document Security*) requires ASes to weigh the [draft §6](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00#name-security-considerations) implications of taking a URL from an unknown client and fetching it:

- **[[server-side-request-forgery|SSRF]] (Authorization Server Abuse Protection).** The AS fetches an attacker-supplied URL and could be steered into requesting internal/admin endpoints. ASes **SHOULD** consider SSRF risks on the metadata fetch.
- **`localhost` redirect-URI impersonation.** CIMD cannot by itself stop an attacker from claiming a legitimate client's metadata URL as their `client_id` and binding a `localhost` port as the `redirect_uri` to capture the code — the user still sees the legitimate `client_name`. ASes **SHOULD** warn on `localhost`-only redirect URIs, **MAY** require additional attestation, and **MUST** clearly display the redirect URI hostname during authorization. (See [[open-redirection]].)
- **Trust policies.** ASes **MAY** apply domain-based policies — allowlists, accept-any-HTTPS for open servers, reputation checks, domain-age/certificate restrictions, prominent hostname display — while retaining full control over access policy.

These are detailed on [[security-considerations]] and feed the [[client-registration]] validation rules.

## Link

- Draft: <https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00>

## Notes

Early IETF draft (`-00`); details are subject to change.
