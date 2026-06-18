---
title: Authorization Server Discovery
category: topic
status: evolving
confidence: high
aliases: [AS discovery, authorization server metadata discovery, protected resource metadata discovery]
enterprise_analogs: [RFC 8414 Authorization Server Metadata, RFC 9728 Protected Resource Metadata, OpenID Connect Discovery 1.0, RFC 9207 issuer identification]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, rfc-9728-protected-resource-metadata, rfc-8414-authorization-server-metadata, openid-connect-discovery, rfc-9207-authorization-server-issuer-identification, authorization-server-mix-up, client-registration]
tags: [oauth, discovery, metadata, mcp]
---

# Authorization Server Discovery

**Authorization server discovery** is how an [[mcp-authorization|MCP client]] — starting from nothing but a server URL — finds the authorization server it must use and learns that AS's endpoints and capabilities. It is the entry point of the whole flow and the reason a general-purpose client can connect to a server it has never seen.

> **Status: evolving.** This page summarizes what the [[mcp-authorization-overview|overview]] states. The dedicated specification document (`raw/MCPAuthorization_AuthorizationServerDiscovery.md`) is present but **not yet ingested**; normative detail will be added then.

## Two-stage discovery (per the overview)

1. **Resource → authorization server.** MCP servers **MUST** implement [[rfc-9728-protected-resource-metadata|Protected Resource Metadata (RFC 9728)]] and advertise it via the `WWW-Authenticate` header's `resource_metadata` on a `401`. Clients **MUST** fetch that metadata and extract the associated authorization server(s), then determine which AS to use.
2. **Authorization server → endpoints.** MCP authorization servers **MUST** provide at least one of [[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata (RFC 8414)]] or [[openid-connect-discovery|OpenID Connect Discovery 1.0]]. Clients **MUST** support **both** mechanisms and try the discovery endpoints in priority order (OAuth 2.0, then OIDC).

The client **records the AS `issuer`** from the validated metadata for later [[rfc-9207-authorization-server-issuer-identification|RFC 9207 `iss` validation]] — the discovery step is where the trusted issuer identity is pinned, defending against [[authorization-server-mix-up]].

## Relation to pre-AI IAM

This is standard OAuth/OIDC metadata discovery: `.well-known/oauth-authorization-server` (RFC 8414), `.well-known/openid-configuration` (OIDC Discovery), and the newer `.well-known/oauth-protected-resource` (RFC 9728). A practitioner who has configured an OIDC relying party against a discovery document already knows the mechanics.

## Why pre-AI IAM is insufficient

In classic deployments the AS is **configured once, by hand**, and rarely changes; discovery is a convenience. For MCP it is **load-bearing**: the client meets servers (and their authorization servers) dynamically at runtime, so RFC 9728 is mandatory on every server and clients must support the full discovery chain plus issuer pinning to avoid being misdirected. Discovery is what makes registration-free, integration-free connection possible — and what makes [[authorization-server-mix-up|mix-up defense]] necessary.
