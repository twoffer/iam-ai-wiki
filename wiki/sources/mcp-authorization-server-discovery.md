---
title: MCP Authorization Specification — Authorization Server Discovery
category: source
status: stable
confidence: high
aliases: ["MCP Authorization Server Discovery", "MCP AS discovery spec", "authorization server discovery spec"]
enterprise_analogs: ["RFC 9728 Protected Resource Metadata", "RFC 9728 §5.1 WWW-Authenticate", "RFC 9728 §7.6 Authorization Servers selection", "RFC 8414 §3.1 Authorization Server Metadata Request", "RFC 8414 §3.3 Validation", "RFC 8414 §5 Compatibility Notes", "OpenID Connect Discovery 1.0 §4.3", "RFC 6749 §2.2 client identifier uniqueness"]
last_updated: 2026-06-19
sources: ["raw/MCPAuthorization_AuthorizationServerDiscovery.md"]
related: ["mcp-authorization", "mcp-authorization-overview", "authorization-server-discovery", "rfc-9728-protected-resource-metadata", "rfc-8414-authorization-server-metadata", "openid-connect-discovery", "authorization-server-mix-up", "client-registration", "mcp-authorization-security-considerations"]
tags: ["mcp", "oauth", "authorization", "discovery", "spec", "source"]
---

# MCP Authorization Specification — Authorization Server Discovery

Summary of `raw/MCPAuthorization_AuthorizationServerDiscovery.md`, pulled 2026-06-17 from <https://modelcontextprotocol.io/specification/draft/basic/authorization/authorization-server-discovery> (the `draft` revision of the Model Context Protocol). This is the second of four documents in the MCP Authorization specification; the others are the [[mcp-authorization-overview|overview]], [[client-registration]], and [[security-considerations]].

## What the document is

It specifies the concrete mechanics by which MCP servers advertise their authorization server(s) and by which MCP clients — starting from only a server URL — discover the authorization server's endpoints and capabilities. It defines no new protocol primitives: it is a tightly constrained profile of [[rfc-9728-protected-resource-metadata|RFC 9728]], [[rfc-8414-authorization-server-metadata|RFC 8414]], and [[openid-connect-discovery|OpenID Connect Discovery 1.0]]. See the topic page [[authorization-server-discovery]] for the synthesized model.

## Key normative claims

### Authorization server location

- MCP servers **MUST** implement [[rfc-9728-protected-resource-metadata|Protected Resource Metadata (RFC 9728)]] to indicate authorization-server locations, and the returned document **MUST** include an `authorization_servers` field listing **at least one** authorization server (*Authorization Server Location*).
- A Protected Resource Metadata document **can list multiple** authorization servers. The responsibility for **selecting which AS to use** lies with the **MCP client**, following [RFC 9728 §7.6](https://datatracker.ietf.org/doc/html/rfc9728#name-authorization-servers).
- When multiple ASes are listed, **each is an independent OAuth 2.0 authorization server**. Per [RFC 6749 §2.2](https://datatracker.ietf.org/doc/html/rfc6749#section-2.2), client identifiers are unique to the AS that issued them. Clients **MUST** maintain **separate registration state (client credentials, tokens) per authorization server** and **MUST NOT** assume credentials valid for one AS are accepted by another. The requirements for associating client credentials with the issuing AS live in *Authorization Server Binding* in the [[client-registration]] document.

### Protected Resource Metadata discovery requirements

MCP servers **MUST** implement **one** of two mechanisms to convey the resource-metadata location (*Protected Resource Metadata Discovery Requirements*):

1. **`WWW-Authenticate` header** — include the resource-metadata URL under `resource_metadata` in the `WWW-Authenticate` header of a `401 Unauthorized` response ([RFC 9728 §5.1](https://datatracker.ietf.org/doc/html/rfc9728#name-www-authenticate-response)).
2. **Well-known URI** — serve the metadata per RFC 9728 either at the path of the MCP endpoint (e.g., MCP at `https://example.com/public/mcp` → metadata at `https://example.com/.well-known/oauth-protected-resource/public/mcp`) or at the root (`https://example.com/.well-known/oauth-protected-resource`).

MCP clients **MUST** support **both** mechanisms: use the `resource_metadata` URL from a parsed `WWW-Authenticate` header when present; otherwise **MUST** fall back to constructing and requesting the well-known URIs in the order above (path-based first, then root). Clients **MUST** be able to parse `WWW-Authenticate` headers and respond to `401`. Servers **may** also include a `scope` parameter in the challenge; its semantics are defined in the [[scope-selection-strategy|scope selection strategy]].

### Authorization server metadata discovery

- MCP uses the default `oauth-authorization-server` well-known URI suffix from [RFC 8414 §3.1](https://datatracker.ietf.org/doc/html/rfc8414#section-3.1); it defines **no** MCP-specific suffix.
- For interoperability with both OAuth 2.0 AS Metadata and OIDC Discovery 1.0, clients **MUST** attempt multiple well-known endpoints, drawing on RFC 8414 §3.1 and the §5 compatibility notes.
- **Issuer URL with a path component** (e.g., `https://auth.example.com/tenant1`) — try, in priority order:
  1. `https://auth.example.com/.well-known/oauth-authorization-server/tenant1` (OAuth 2.0, path insertion)
  2. `https://auth.example.com/.well-known/openid-configuration/tenant1` (OIDC, path insertion)
  3. `https://auth.example.com/tenant1/.well-known/openid-configuration` (OIDC, path appending)
- **Issuer URL without a path component** (e.g., `https://auth.example.com`) — try, in priority order:
  1. `https://auth.example.com/.well-known/oauth-authorization-server` (OAuth 2.0)
  2. `https://auth.example.com/.well-known/openid-configuration` (OIDC)

### Metadata validation

After retrieving a metadata document, clients **MUST** validate it per [RFC 8414 §3.3](https://datatracker.ietf.org/doc/html/rfc8414#section-3.3) or [OIDC Discovery §4.3](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationValidation): the `issuer` value in the document **MUST** be identical to the issuer identifier used to construct the well-known URL. If they differ, the client **MUST NOT** use the metadata. The spec's worked example: a document fetched from `https://attacker.example/.well-known/oauth-authorization-server` carrying `"issuer": "https://honest.example"` **MUST** be rejected. This is a discovery-layer defense against [[authorization-server-mix-up]], distinct from the [[rfc-9207-authorization-server-issuer-identification|RFC 9207]] validation of the `iss` in the authorization *response*.

### Example flow (sequence diagram)

Unauthenticated MCP request → `401` (may carry `WWW-Authenticate`) → if the header carries `resource_metadata`, GET it; otherwise probe the well-known URIs (sub-path, then root; the sub-path step is "not applicable if the MCP server is at the root") → validate resource-server metadata and build the AS-metadata URL → GET the AS-metadata endpoint, trying OAuth 2.0 then OIDC in priority order → the OAuth 2.1 authorization flow → token request → access token → MCP request with the bearer token.

## Relation to pre-AI IAM

The endpoint construction is ordinary OAuth/OIDC: `.well-known/oauth-protected-resource` (RFC 9728), `.well-known/oauth-authorization-server` (RFC 8414), and `.well-known/openid-configuration` (OIDC Discovery), with the RFC 8414 §5 path-insertion/path-appending variants a federation engineer already handles. The document's contribution beyond the base specs is to (a) make **client-side** support of *all* the probing variants and *both* metadata flavors mandatory, (b) require **per-authorization-server isolation** of client credentials and tokens (RFC 6749 §2.2), and (c) mandate **metadata-document issuer validation** so a client meeting a server at runtime cannot be steered to an impostor AS. Those are precisely the controls a human integrator used to supply by hand when wiring a single, known AS. See [[authorization-server-discovery]] for the gap analysis.
