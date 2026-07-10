---
title: Client Registration
category: topic
status: stable
confidence: high
aliases: [MCP client registration, client ID metadata documents, dynamic client registration, DCR]
enterprise_analogs: [RFC 7591 Dynamic Client Registration, OpenID Connect Dynamic Client Registration 1.0 application_type, manual client pre-registration, OAuth Client ID Metadata Documents draft-ietf-oauth-client-id-metadata-document-00, RFC 6749 §2.2 client identifier uniqueness]
last_updated: 2026-07-08
sources: [mcp-authorization-client-registration, mcp-authorization-server-discovery, mcp-authorization-overview, mcp-authorization-security-considerations, mcp-security-best-practices]
related: [mcp-authorization, oauth-client-id-metadata-documents, rfc-7591-dynamic-client-registration, openid-connect-dynamic-client-registration, rfc-8414-authorization-server-metadata, authorization-server-discovery, security-considerations, server-side-request-forgery, confused-deputy, public-vs-confidential-client]
tags: [oauth, client-registration, cimd, dcr, mcp]
---

# Client Registration

**Client registration** is how an [[mcp-authorization|MCP client]] obtains a `client_id` to use with an authorization server. Because MCP clients routinely connect to servers with which they have **no prior relationship**, the spec defines three mechanisms and a selection priority among them ([[mcp-authorization-client-registration]]).

## The three mechanisms

| Mechanism | When to use | Normative level |
| --- | --- | --- |
| **[[oauth-client-id-metadata-documents|Client ID Metadata Documents (CIMD)]]** | Client and server have no prior relationship (the most common MCP case) | AS and clients **SHOULD** support |
| **Pre-registration** | Client and server have an existing relationship | Clients **SHOULD** support static credentials |
| **[[rfc-7591-dynamic-client-registration|Dynamic Client Registration (DCR)]]** | Backwards compatibility or specific requirements | **MAY** support; **deprecated** |

## Selection priority

A client that supports all options **SHOULD** choose in this order ([[mcp-authorization-client-registration]]):

1. **Pre-registered** information, if the client already has it for that server.
2. **Client ID Metadata Documents**, if the AS advertises `client_id_metadata_document_supported` in its [[rfc-8414-authorization-server-metadata|Authorization Server Metadata]].
3. **Dynamic Client Registration**, as a fallback, if the AS exposes a `registration_endpoint`.
4. **Prompt the user** to enter client information, if nothing else is available.

Client identity MUST be established by one of these before the authorization flow begins.

## Client ID Metadata Documents

The preferred mechanism. The client uses an **HTTPS URL as its `client_id`**; the AS dereferences that URL to fetch a JSON metadata document and validates it — no registration call. Both sides **SHOULD** support it per [draft-ietf-oauth-client-id-metadata-document-00](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00). The spec highlights these requirements:

**Client side:**

- Host the metadata document at an HTTPS URL meeting the RFC's requirements.
- The `client_id` URL **MUST** use the `https` scheme and **contain a path component** (e.g., `https://example.com/client.json`).
- The document **MUST** include at least `client_id`, `client_name`, and `redirect_uris`.
- The `client_id` in the document **MUST** match the document URL exactly.
- **MAY** authenticate with `private_key_jwt` (with JWKS) at the token endpoint (draft §6.2) — otherwise the client is public (`token_endpoint_auth_method: "none"`).

**Authorization-server side:**

- **SHOULD** fetch the document on encountering a URL-formatted `client_id`.
- **MUST** validate that the fetched document's `client_id` equals the URL.
- **MUST** validate the request's `redirect_uri` against the document's `redirect_uris`.
- **MUST** validate that the document is valid JSON with the required fields.
- **SHOULD** cache the document, respecting HTTP cache headers.
- **SHOULD** follow the draft §6 security considerations and the *Client ID Metadata Document Security* rules in [[security-considerations]] (e.g., optionally gating fetched domains behind a trust policy). On failure the AS returns `invalid_client` or `invalid_request`.

The AS advertises support with `"client_id_metadata_document_supported": true` in its metadata; clients **SHOULD** check it and **MAY** fall back to DCR or pre-registration if absent.

## Pre-registration

Clients **SHOULD** support static client credentials. Two forms ([[mcp-authorization-client-registration]]):

1. A **hardcoded** client ID (and credentials, if any) for a specific authorization server.
2. A **UI** that lets the user paste in details after they register an OAuth client themselves — e.g., through a configuration interface the server hosts.

## Dynamic Client Registration (deprecated)

[[rfc-7591-dynamic-client-registration|RFC 7591]] lets a client `POST` metadata to a `/register` endpoint and obtain a `client_id` with no user interaction. The MCP profile **deprecates** it in favor of CIMD, retaining it only for backwards compatibility with authorization servers that do not support CIMD.

**Application type and redirect URIs.** When an AS supports both OIDC and DCR, it may constrain redirect URIs by the [[openid-connect-dynamic-client-registration|OIDC]] `application_type`. Clients **MUST** set an appropriate value — omitting it defaults to `"web"` under OIDC, which can conflict with native-style redirect URIs (non-OIDC servers ignore it):

- **`native`** — desktop apps, mobile apps, CLI tools, and `localhost`-hosted web apps.
- **`web`** — remote browser-based apps served from a non-local host.

Clients **MUST** handle registration rejections caused by redirect-URI constraints, **SHOULD** surface a meaningful error, and **MAY** retry with an adjusted `application_type` or conforming redirect URIs.

## Authorization Server Binding (per-AS state)

A single MCP server may front **multiple, independent** authorization servers ([[authorization-server-discovery]]), and a `client_id` is unique to the AS that issued it ([RFC 6749 §2.2](https://datatracker.ietf.org/doc/html/rfc6749#section-2.2)). The binding rules ([[mcp-authorization-client-registration]], *Authorization Server Binding*):

- Clients that use pre-registered credentials, or persist DCR-obtained credentials, **MUST** associate each credential with its issuing AS, keyed by the AS `issuer` identifier.
- When the AS changes — detected via updated [[rfc-9728-protected-resource-metadata|protected resource metadata]] — clients **MUST NOT** reuse credentials from a different AS and **MUST** re-register with the new one.
- If the AS indicated by protected resource metadata no longer matches the one a pre-registered credential belongs to, clients **SHOULD** surface an error rather than silently using mismatched credentials.
- **CIMD client IDs are exempt and portable.** Because they are self-hosted HTTPS URLs the AS resolves on demand, no re-registration is needed when the AS changes.

This is the registration-state counterpart to the per-AS credential isolation that [[authorization-server-discovery]] requires when a resource lists several authorization servers.

## Security of the registration mechanisms

Each registration path carries threats the [[mcp-authorization-security-considerations|Security Considerations]] document treats normatively ([[security-considerations]]):

- **CIMD fetch → [[server-side-request-forgery|SSRF]].** Because the AS dereferences a `client_id` URL supplied by an unknown client, it can be steered into requesting internal endpoints; ASes SHOULD apply SSRF defenses and the trust policies above.
- **CIMD `localhost` impersonation.** Exact `redirect_uri` matching cannot distinguish a legitimate `localhost` client from an attacker who claims the same metadata URL; ASes MUST display the redirect-URI hostname and SHOULD warn on `localhost`-only URIs (see [[open-redirection]]).
- **Proxy servers → [[confused-deputy]].** An MCP proxy with a **static** upstream client ID MUST obtain user consent for **each dynamically registered client** before forwarding to a third-party AS, so prior consent on the static ID cannot be ridden by a malicious downstream client. Unauthenticated dynamic registration is one of the four vulnerable conditions in the attack's full anatomy ([[mcp-security-best-practices]]): it is what lets the attacker mint a client whose self-declared `redirect_uri` points at attacker infrastructure. See [[confused-deputy]] for the complete condition list and mitigation stack (per-client consent registry, consent UI, cookie hygiene, `state` lifecycle).

## Relation to pre-AI IAM

The three mechanisms span the history of OAuth client identity. **Pre-registration** is the classic enterprise model — an admin enrolls each app and hands it a client ID/secret. **DCR** ([[rfc-7591-dynamic-client-registration|RFC 7591]] / [[openid-connect-dynamic-client-registration|OIDC Dynamic Client Registration]]) automated that for federations. **CIMD** is the newest turn: a publicly dereferenceable URL *is* the identity, akin in spirit to identifying a SAML entity by its metadata URL. *Authorization Server Binding* is RFC 6749 §2.2 made explicit — client identifiers are unique to their issuing AS — and the `application_type` rules are ordinary OIDC registration semantics a federation engineer already applies.

## Why pre-AI IAM is insufficient

Manual pre-registration assumes a **bounded, known set of clients** an administrator can enroll ahead of time. An MCP ecosystem has an **open, unbounded client population** meeting **open, unbounded servers** — no admin can pre-register every agent against every server. The profile's answer is registration-free identity: Client ID Metadata Documents let a client present a URL the AS resolves on the spot, and it explicitly **deprecates** classic Dynamic Client Registration (whose unauthenticated `/register` endpoint is itself an abuse surface) in favor of it. Even the surviving credential-bearing paths are hardened for the open model: any persisted credential is pinned to its issuing `issuer`, so a client fronted by several independent authorization servers cannot misapply a credential across them. This is the registration analog of the same shift seen in [[authorization-server-discovery]]: replace human-time integration with runtime, self-describing discovery.
