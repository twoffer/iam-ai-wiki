---
title: MCP Authorization Specification — Client Registration
category: source
status: stable
confidence: high
aliases: [MCP Client Registration, MCP client registration spec, MCP client ID metadata documents]
enterprise_analogs: [OAuth Client ID Metadata Document draft-ietf-oauth-client-id-metadata-document-00, draft-ietf-oauth-client-id-metadata-document-00 §6.2 private_key_jwt, RFC 7591 Dynamic Client Registration, OpenID Connect Dynamic Client Registration 1.0 application_type, RFC 6749 §2.2 client identifier uniqueness, private_key_jwt client authentication]
last_updated: 2026-06-19
sources: [raw/MCPAuthorization_ClientRegistration.md]
related: [mcp-authorization, mcp-authorization-overview, client-registration, oauth-client-id-metadata-documents, rfc-7591-dynamic-client-registration, openid-connect-dynamic-client-registration, rfc-8414-authorization-server-metadata, authorization-server-discovery, security-considerations, mcp-authorization-security-considerations]
tags: [mcp, oauth, client-registration, cimd, dcr, spec, source]
---

# MCP Authorization Specification — Client Registration

Summary of `raw/MCPAuthorization_ClientRegistration.md`, pulled 2026-06-17 from <https://modelcontextprotocol.io/specification/draft/basic/authorization/client-registration> (the `draft` revision of the Model Context Protocol). This is the third of four documents in the MCP Authorization specification; the others are the [[mcp-authorization-overview|overview]], [[authorization-server-discovery|Authorization Server Discovery]], and [[security-considerations]].

## What the document is

It specifies how an [[mcp-authorization|MCP client]] obtains a `client_id` to present to an authorization server, given the defining MCP condition that client and server often have **no prior relationship**. It defines no new protocol: it composes three existing mechanisms — [[oauth-client-id-metadata-documents|OAuth Client ID Metadata Documents]] (an IETF draft), classic pre-registration, and [[rfc-7591-dynamic-client-registration|RFC 7591 Dynamic Client Registration]] — assigns them a selection priority, and adds rules for binding persisted credentials to the issuing authorization server. See the topic page [[client-registration]] for the synthesized model.

## Key normative claims

### Three mechanisms and a selection priority

MCP defines three registration mechanisms, chosen by scenario:

- **[[oauth-client-id-metadata-documents|Client ID Metadata Documents (CIMD)]]** — when client and server have no prior relationship (described as the most common MCP case).
- **Pre-registration** — when client and server already have a relationship.
- **[[rfc-7591-dynamic-client-registration|Dynamic Client Registration (DCR)]]** — for backwards compatibility or specific requirements.

Clients that support all options **SHOULD** apply this priority order:

1. Use **pre-registered** client information if the client already has it for that server.
2. Use **Client ID Metadata Documents** if the AS advertises support (`client_id_metadata_document_supported` in [[rfc-8414-authorization-server-metadata|OAuth Authorization Server Metadata]]).
3. Use **Dynamic Client Registration** as a fallback if the AS supports it (`registration_endpoint` present in AS metadata).
4. **Prompt the user** to enter client information if no other option is available.

### Client ID Metadata Documents

Both MCP clients and authorization servers **SHOULD** support CIMD as specified in [draft-ietf-oauth-client-id-metadata-document-00](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00). The approach lets a client use an **HTTPS URL as its `client_id`**, where the URL dereferences to a JSON metadata document. Implementations supporting CIMD **MUST** follow the draft's requirements; the document calls out:

**For MCP clients:**

- **MUST** host the metadata document at an HTTPS URL meeting the RFC's requirements.
- The `client_id` URL **MUST** use the `https` scheme and **contain a path component** (e.g., `https://example.com/client.json`).
- The metadata document **MUST** include at least `client_id`, `client_name`, and `redirect_uris`.
- The `client_id` value inside the metadata **MUST** match the document URL exactly.
- **MAY** use `private_key_jwt` for client authentication (e.g., at the token endpoint) with appropriate JWKS configuration ([§6.2 of the draft](https://www.ietf.org/archive/id/draft-ietf-oauth-client-id-metadata-document-00.html#section-6.2)).

**For authorization servers:**

- **SHOULD** fetch the metadata document when it encounters a URL-formatted `client_id`.
- **MUST** validate that the fetched document's `client_id` matches the URL exactly.
- **SHOULD** cache the metadata, respecting HTTP cache headers.
- **MUST** validate the `redirect_uris` presented in an authorization request against those in the metadata document.
- **MUST** validate that the document is valid JSON and contains the required fields.
- **SHOULD** follow the security considerations in [§6 of the draft](https://www.ietf.org/archive/id/draft-ietf-oauth-client-id-metadata-document-00.html#section-6) and the *Client ID Metadata Document Security* section of [[security-considerations]].

Example metadata document (verbatim from the spec):

```json
{
  "client_id": "https://app.example.com/oauth/client-metadata.json",
  "client_name": "Example MCP Client",
  "client_uri": "https://app.example.com",
  "logo_uri": "https://app.example.com/logo.png",
  "redirect_uris": [
    "http://127.0.0.1:3000/callback",
    "http://localhost:3000/callback"
  ],
  "grant_types": ["authorization_code"],
  "response_types": ["code"],
  "token_endpoint_auth_method": "none"
}
```

The flow (sequence diagram in the doc): the client sends an authorization request with `client_id` set to its HTTPS metadata URL; the AS authenticates the user, detects the URL-formatted `client_id`, `GET`s the metadata document, validates (client_id matches URL, redirect_uri in the allowed list, document structure valid, and optionally the domain against a trust policy), then on success shows a consent page using `client_name`, redirects with the authorization code, exchanges the code for a token (again presenting the URL `client_id`), and caches the metadata for future requests. On validation failure it returns `error=invalid_client` or `invalid_request`.

**Advertising CIMD support.** Authorization servers signal support with `"client_id_metadata_document_supported": true` in their OAuth Authorization Server Metadata. Clients **SHOULD** check for this capability and **MAY** fall back to DCR or pre-registration if it is absent.

### Pre-registration

Clients **SHOULD** support static client credentials such as those supplied by a pre-registration flow, in either of two forms: (1) a hardcoded client ID (and credentials, if applicable) for use with a specific AS, or (2) a UI through which the user enters details after registering an OAuth client themselves (e.g., via a configuration interface the server hosts).

### Dynamic Client Registration (deprecated)

DCR is **deprecated**; new implementations should use CIMD instead. It remains available for backwards compatibility with authorization servers that do not support CIMD. Clients and authorization servers **MAY** support [[rfc-7591-dynamic-client-registration|RFC 7591]] to let clients obtain client IDs without user interaction.

**Application type and redirect-URI constraints.** When an AS supports OIDC and DCR, it may constrain redirect URIs based on the `application_type` parameter from [OpenID Connect Dynamic Client Registration 1.0](https://openid.net/specs/openid-connect-registration-1_0.html). Clients **MUST** specify an appropriate `application_type`: omitting it defaults to `"web"` under OIDC, which can conflict with native-style redirect URIs (non-OIDC servers ignore the parameter safely).

- **Native applications** — desktop apps, mobile apps, CLI tools, and locally-hosted web apps reached via `localhost` — **SHOULD** use `application_type: "native"`.
- **Web applications** — remote browser-based apps served from a non-local host — **SHOULD** use `application_type: "web"`.

Clients **MUST** be prepared to handle registration failures caused by redirect-URI constraints when the AS implements OIDC, **SHOULD** surface a meaningful error to the user or developer, and **MAY** retry with an adjusted `application_type` or with redirect URIs that conform to the AS's requirements for that type.

### Authorization Server Binding

Clients that use pre-registered credentials, or that persist credentials obtained via DCR, **MUST** associate those credentials with the specific authorization server that issued them, keyed by the AS's `issuer` identifier. When the authorization server changes — detected via updated [[rfc-9728-protected-resource-metadata|protected resource metadata]] ([[authorization-server-discovery]]) — clients **MUST NOT** reuse credentials from a different AS and **MUST** re-register with the new one.

Pre-registered credentials are inherently specific to a particular AS; if the AS indicated by protected resource metadata no longer matches the one the credentials were registered with, clients **SHOULD** surface an error rather than silently attempting to use mismatched credentials.

Client IDs based on Client ID Metadata Documents are **portable** across authorization servers, because they are self-hosted HTTPS URLs the AS resolves on demand — no re-registration is needed when the AS changes.

## Relation to pre-AI IAM

The three mechanisms map onto the full history of how OAuth clients get identities. **Pre-registration** is the classic enterprise model: an administrator enrolls each application out of band and hands it a `client_id` (and secret). **DCR** ([[rfc-7591-dynamic-client-registration|RFC 7591]], and the [[openid-connect-dynamic-client-registration|OIDC profile]]) automated that enrollment for federations via a `/register` endpoint. **CIMD** is the newest turn: a publicly dereferenceable HTTPS URL *is* the identity, conceptually similar to how a SAML entity is named by its metadata URL. The `application_type` rules are ordinary OIDC Dynamic Client Registration semantics, and *Authorization Server Binding* restates RFC 6749 §2.2 (client identifiers are unique to their issuing AS) as an explicit per-issuer keying and re-registration requirement.

The document's distinct contribution is to make those choices safe for an **open, unbounded** population of clients meeting an **open, unbounded** population of servers, where no human pre-integrates the pair. Its answers: prefer registration-free, self-hosted identity (CIMD); **deprecate** classic DCR (whose unauthenticated `/register` endpoint is itself an abuse surface) to backwards-compatibility-only; and pin any persisted credential to its issuing AS so a client fronted by several independent authorization servers ([[authorization-server-discovery]]) cannot leak or misapply a credential across them. See [[client-registration]] for the gap analysis.
