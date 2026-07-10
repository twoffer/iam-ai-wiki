---
title: Authorization Server Discovery
category: topic
status: stable
confidence: high
aliases: ["AS discovery", "authorization server metadata discovery", "protected resource metadata discovery"]
enterprise_analogs: ["RFC 8414 Authorization Server Metadata", "RFC 9728 Protected Resource Metadata", "OpenID Connect Discovery 1.0", "RFC 9207 issuer identification", "RFC 6749 §2.2 client identifier uniqueness"]
last_updated: 2026-06-19
sources: ["mcp-authorization-server-discovery", "mcp-authorization-overview"]
related: ["mcp-authorization", "rfc-9728-protected-resource-metadata", "rfc-8414-authorization-server-metadata", "openid-connect-discovery", "rfc-9207-authorization-server-issuer-identification", "authorization-server-mix-up", "client-registration"]
tags: ["oauth", "discovery", "metadata", "mcp"]
---

# Authorization Server Discovery

**Authorization server discovery** is how an [[mcp-authorization|MCP client]] — starting from nothing but a server URL — finds the authorization server it must use and learns that AS's endpoints and capabilities. It is the entry point of the whole flow and the reason a general-purpose client can connect to a server it has never seen. The mechanics below are specified in the [[mcp-authorization-server-discovery|Authorization Server Discovery]] document (doc 2 of the spec); the [[mcp-authorization-overview|overview]] introduces the two stages.

Discovery is **two stages**: first resource → authorization server (which AS does this MCP server trust?), then authorization server → endpoints (where are that AS's authorization and token endpoints?).

## Stage 1 — Locating the authorization server

The MCP server (an OAuth resource server) **MUST** implement [[rfc-9728-protected-resource-metadata|Protected Resource Metadata (RFC 9728)]], and its metadata document **MUST** carry an `authorization_servers` field listing **at least one** authorization server.

A single resource can list **multiple** authorization servers, each an **independent** OAuth 2.0 AS. Two consequences ([[mcp-authorization-server-discovery]], *Authorization Server Location*):

- **The client selects the AS.** When several are listed, choosing which to use is the client's responsibility, per [RFC 9728 §7.6](https://datatracker.ietf.org/doc/html/rfc9728#name-authorization-servers).
- **State is isolated per AS.** Because client identifiers are unique to the AS that issued them ([RFC 6749 §2.2](https://datatracker.ietf.org/doc/html/rfc6749#section-2.2)), clients **MUST** keep **separate registration state — client credentials and tokens — per authorization server** and **MUST NOT** assume credentials valid at one AS are accepted at another. The binding requirements live in *Authorization Server Binding* in the [[client-registration]] document.

### How the resource-metadata location reaches the client

The server **MUST** implement at least one of two mechanisms; the client **MUST** support **both**:

| Mechanism | Server behavior | Client behavior |
| --- | --- | --- |
| **`WWW-Authenticate` header** | On `401`, include the metadata URL under `resource_metadata` ([RFC 9728 §5.1](https://datatracker.ietf.org/doc/html/rfc9728#name-www-authenticate-response)) | Parse the header and use the `resource_metadata` URL when present |
| **Well-known URI** | Serve metadata at the path-based URI (MCP at `https://example.com/public/mcp` → `https://example.com/.well-known/oauth-protected-resource/public/mcp`) or at the root (`https://example.com/.well-known/oauth-protected-resource`) | When no `resource_metadata` header is present, **fall back** to constructing/requesting the well-known URIs — path-based first, then root |

The server may also include a `scope` parameter in the `WWW-Authenticate` challenge; its semantics belong to the [[scope-selection-strategy|scope selection strategy]] (via [[rfc-6750-bearer-token-usage|RFC 6750]]).

## Stage 2 — Discovering the authorization server's endpoints

The AS **MUST** provide at least one of [[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata (RFC 8414)]] or [[openid-connect-discovery|OpenID Connect Discovery 1.0]]; the client **MUST** support **both** and **MUST** probe multiple well-known endpoints to cover the different issuer-URL shapes. MCP uses the default `oauth-authorization-server` suffix from [RFC 8414 §3.1](https://datatracker.ietf.org/doc/html/rfc8414#section-3.1) and defines **no** MCP-specific suffix; the OIDC interoperability comes from RFC 8414 §5.

**If the issuer URL has a path component** (e.g., `https://auth.example.com/tenant1`), probe in this priority order:

1. `https://auth.example.com/.well-known/oauth-authorization-server/tenant1` — OAuth 2.0, **path insertion**
2. `https://auth.example.com/.well-known/openid-configuration/tenant1` — OIDC, **path insertion**
3. `https://auth.example.com/tenant1/.well-known/openid-configuration` — OIDC, **path appending**

**If the issuer URL has no path component** (e.g., `https://auth.example.com`), probe in this priority order:

1. `https://auth.example.com/.well-known/oauth-authorization-server` — OAuth 2.0
2. `https://auth.example.com/.well-known/openid-configuration` — OIDC

In all cases OAuth 2.0 AS Metadata is tried before OIDC Discovery, and (for path issuers) `.well-known`-insertion is tried before path-appending.

## Metadata validation — two issuer checks

Discovery pins the trusted issuer identity, and the spec layers **two distinct** issuer validations; conflating them is a common error:

1. **Metadata-document issuer match (this document).** After fetching an AS-metadata document, the client **MUST** validate it per [RFC 8414 §3.3](https://datatracker.ietf.org/doc/html/rfc8414#section-3.3) / [OIDC Discovery §4.3](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationValidation): the `issuer` inside the document **MUST** be identical to the issuer identifier used to build the well-known URL. A document fetched from `https://attacker.example/.well-known/oauth-authorization-server` that claims `"issuer": "https://honest.example"` **MUST** be rejected.
2. **Authorization-response `iss` validation ([[mcp-authorization-overview|overview]] doc).** Later, the client records the validated `issuer` and compares the `iss` returned in the authorization response against it via exact string comparison ([[rfc-9207-authorization-server-issuer-identification|RFC 9207]]).

The first protects the **discovery** step (you fetched metadata from the right place); the second protects the **redirect/response** step (the response came from the AS you expected). Together they defend against [[authorization-server-mix-up]]; mix-up defense also composes with [[proof-key-for-code-exchange|PKCE]], which protects the authorization code itself.

## Relation to pre-AI IAM

This is standard OAuth/OIDC metadata discovery: `.well-known/oauth-authorization-server` (RFC 8414), `.well-known/openid-configuration` (OIDC Discovery), and the newer `.well-known/oauth-protected-resource` (RFC 9728). The path-insertion vs. path-appending variants are exactly RFC 8414 §5's compatibility rules, and per-AS credential scoping is RFC 6749 §2.2. A practitioner who has configured an OIDC relying party against a discovery document, or run a multi-tenant AS behind path-based issuers, already knows the mechanics.

## Why pre-AI IAM is insufficient

In classic deployments the AS is **configured once, by hand**, and rarely changes; discovery is a convenience and the human integrator supplies the trust decisions. For MCP it is **load-bearing**: the client meets servers (and their authorization servers) dynamically at runtime, so RFC 9728 is mandatory on every server, clients must implement the **entire** probing matrix and **both** metadata flavors, must isolate credentials **per AS** because one resource may front several independent ASes, and must perform **metadata-issuer validation** automatically because there is no human to notice they were handed an impostor endpoint. Discovery is what makes registration-free, integration-free connection possible — and what makes [[authorization-server-mix-up|mix-up defense]] necessary at machine speed.
