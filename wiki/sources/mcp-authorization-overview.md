---
title: MCP Authorization Specification — Overview
category: source
status: stable
confidence: high
aliases: [MCP Authorization Overview, modelcontextprotocol authorization, MCP auth spec]
enterprise_analogs: [OAuth 2.1 draft-ietf-oauth-v2-1-13, RFC 6750 Bearer Token Usage, RFC 8414 Authorization Server Metadata, RFC 7591 Dynamic Client Registration, RFC 8707 Resource Indicators, RFC 9728 Protected Resource Metadata, RFC 9207 Authorization Server Issuer Identification]
last_updated: 2026-06-18
sources: [raw/MCPAuthorization_Overview.md]
related: [mcp-authorization, mcp-specification, oauth-2-1, authorization-server-discovery, client-registration, scope-selection-strategy, canonical-server-uri, security-considerations]
tags: [mcp, oauth, authorization, spec, source]
---

# MCP Authorization Specification — Overview

Summary of `raw/MCPAuthorization_Overview.md`, pulled 2026-06-17 from <https://modelcontextprotocol.io/specification/draft/basic/authorization> (the `draft` revision of the Model Context Protocol). This is the first of four documents in the MCP Authorization specification; the others are [[authorization-server-discovery]], [[client-registration]], and [[security-considerations]].

## What the document is

The overview defines MCP's transport-level authorization model: how an [[mcp-authorization|MCP client]] obtains and presents OAuth tokens to a protected MCP server "on behalf of resource owners" (*Introduction*). It is a **profile of OAuth 2.1** — it selects a subset of existing OAuth/OIDC specifications rather than inventing new mechanisms (*Standards Compliance*).

## Key normative claims

- **Authorization is OPTIONAL.** HTTP-based transports SHOULD conform to this spec; STDIO transports SHOULD NOT — they retrieve credentials from the environment instead; other transports MUST follow their own best practices (*Protocol Requirements*). See [[machine-identity]] for the STDIO/environment path.
- **Roles** (*Roles*): the MCP server is an OAuth 2.1 **resource server**; the MCP client is an OAuth 2.1 **client**; a separate **authorization server** authenticates the user and issues tokens. The AS may be co-located with the resource server or independent. See [[mcp-authorization]].
- **Standards baseline** (*Standards Compliance / Overview*): authorization servers MUST implement [[oauth-2-1]]. MCP servers MUST implement [[rfc-9728-protected-resource-metadata]] (Protected Resource Metadata); clients MUST use it for [[authorization-server-discovery]]. Authorization servers MUST provide at least one of [[rfc-8414-authorization-server-metadata]] or [[openid-connect-discovery]]; clients MUST support both. Servers and clients SHOULD support [[oauth-client-id-metadata-documents]] and MAY support [[rfc-7591-dynamic-client-registration]] (now deprecated, retained for backwards compatibility). See [[client-registration]].
- **Scope selection** (*Scope Selection Strategy*): servers SHOULD advertise required scopes in the `WWW-Authenticate` `scope` parameter ([[rfc-6750-bearer-token-usage]] §3). Clients MUST treat challenged scopes as authoritative and MUST NOT assume any set relationship to `scopes_supported`. Priority: use the challenge `scope`, else fall back to all of `scopes_supported`. See [[scope-selection-strategy]].
- **Authorization flow** (*Authorization Flow Steps*): unauthenticated request → `401` with `WWW-Authenticate` → fetch Protected Resource Metadata → discover AS → register/identify client → PKCE + `resource` + scope → browser authorization → code callback with `iss` → validate `iss` → token request → bearer token on subsequent MCP requests. Uses [[proof-key-for-code-exchange|PKCE]] and the [[canonical-server-uri|resource parameter]].
- **Authorization response validation** (*Authorization Response Validation*): clients MUST record the AS `issuer` before redirecting and validate the returned `iss` per [[rfc-9207-authorization-server-issuer-identification|RFC 9207]] §2.4, using exact string comparison with no URI normalization. Validation applies to error responses too. AS inclusion of `iss` is SHOULD today, with a stated intent to upgrade to MUST in a future revision. Defends against [[authorization-server-mix-up]].
- **Resource parameter** (*Resource Parameter Implementation*): clients MUST implement [[rfc-8707-resource-indicators|RFC 8707]] — the `resource` parameter MUST appear in both authorization and token requests, MUST identify the target MCP server by its [[canonical-server-uri|canonical URI]], and MUST be sent regardless of AS support.
- **Token usage** (*Access Token Usage*): bearer tokens in the `Authorization` header on every request, never in the query string. Servers MUST validate that tokens were issued for them as the intended audience ([[token-audience-binding]]). Servers MUST NOT accept or transit any other tokens — a prohibition on [[token-passthrough]].
- **Refresh tokens** (*Refresh Tokens*): clients keep them confidential, SHOULD list `refresh_token` in `grant_types`, and MAY request `offline_access`. Servers SHOULD NOT surface `offline_access` in challenges or `scopes_supported`.
- **Error handling & step-up** (*Error Handling*): `401` (auth required / invalid token), `403` (insufficient scope / permission), `400` (malformed). Runtime insufficient-scope returns `403` with `error="insufficient_scope"` and a `scope` challenge; clients perform [[step-up-authorization]], accumulating scopes client-side (servers stay stateless about client scope sets).
- **Security considerations** (*Security Considerations*): defers normative security requirements to [[security-considerations]] — token audience binding/validation, token theft, communication security, authorization code protection, mix-up and [[confused-deputy]] attacks, open redirection, and Client ID Metadata Document security.
- **Extensions** (*MCP Authorization Extensions*): optional, additive, composable, independently versioned extensions exist in the [[mcp-authorization-extensions|ext-auth]] repository.

## Relation to pre-AI IAM

The document's primary contribution is **bridging**: it defines no new cryptographic or protocol primitives. It constrains a general-purpose AI tool-calling client to behave as a disciplined OAuth 2.1 public client, and a tool-exposing server to behave as an OAuth 2.1 resource server with mandatory audience restriction (RFC 8707) and mandatory protected-resource metadata (RFC 9728). For a practitioner with OAuth/OIDC background, almost every mechanism is familiar; what is new is *which* options are mandatory and *why* (see the "Why pre-AI IAM is insufficient" sections on [[mcp-authorization]] and [[token-audience-binding]]).

## Notable for future ingests

- Three sibling documents are referenced and seeded here as forward stubs: [[authorization-server-discovery]], [[client-registration]], [[security-considerations]]. The raw files (`raw/MCPAuthorization_AuthorizationServerDiscovery.md`, `raw/MCPAuthorization_ClientRegistration.md`, `raw/MCPAuthorization_SecurityConsiderations.md`) are present but not yet ingested.
- The spec is a **draft** revision; normative levels (e.g., `iss` SHOULD → MUST) are expected to change.
