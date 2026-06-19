---
title: OpenID Connect Discovery 1.0
category: reference
status: stable
confidence: high
aliases: [OIDC Discovery, OpenID Connect Discovery, openid-configuration, well-known openid]
enterprise_analogs: []
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations, mcp-authorization-server-discovery, mcp-authorization-overview]
related: [authorization-server-discovery, rfc-8414-authorization-server-metadata, openid-connect-dynamic-client-registration, mcp-authorization, proof-key-for-code-exchange, openid-foundation, authorization-server-mix-up]
tags: [oidc, discovery, metadata, spec, openid-foundation, reference]
---

# OpenID Connect Discovery 1.0

**OpenID Connect Discovery 1.0** (from the [[openid-foundation|OpenID Foundation]]) defines the `.well-known/openid-configuration` metadata document by which an OpenID Provider advertises its endpoints and capabilities. It predates and inspired [[rfc-8414-authorization-server-metadata|RFC 8414]], with which it largely overlaps.

## Role in MCP Authorization

The second of the two AS-discovery mechanisms. Per the [[mcp-authorization-overview]], an MCP authorization server MUST provide **at least one** of [[rfc-8414-authorization-server-metadata|RFC 8414]] or OIDC Discovery, and MCP clients MUST support **both**, trying OAuth 2.0 metadata before the OIDC endpoint. See [[authorization-server-discovery]].

The [[mcp-authorization-server-discovery|AS Discovery]] document specifies **two OIDC endpoint shapes** the client must try (after the OAuth 2.0 endpoint) for an issuer with a path component such as `https://auth.example.com/tenant1`:

- **Path insertion:** `https://auth.example.com/.well-known/openid-configuration/tenant1`
- **Path appending:** `https://auth.example.com/tenant1/.well-known/openid-configuration`

For a path-less issuer (`https://auth.example.com`) only `https://auth.example.com/.well-known/openid-configuration` applies. Retrieved documents MUST be validated per [§4.3](https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfigurationValidation): the document's `issuer` MUST be identical to the issuer used to build the URL, or the client rejects it — see [[authorization-server-mix-up]].

## `code_challenge_methods_supported` requirement

The [[mcp-authorization-security-considerations|Security Considerations]] document closes a gap specific to OIDC: the OpenID Provider Metadata schema **does not define** `code_challenge_methods_supported`, even though many providers include it. For MCP, this field is the only [[proof-key-for-code-exchange|PKCE]]-support discovery signal, so the document requires:

- MCP clients MUST **verify the presence** of `code_challenge_methods_supported` in the OIDC provider metadata and MUST **refuse to proceed** if it is absent.
- Authorization servers providing OIDC Discovery 1.0 MUST **include** `code_challenge_methods_supported` to be MCP-compatible.

This makes a non-standard-but-common field effectively mandatory under the MCP profile. The equivalent rule for [[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata]], where the field *is* defined, is on that page.

## Link

- Spec: <https://openid.net/specs/openid-connect-discovery-1_0.html>
