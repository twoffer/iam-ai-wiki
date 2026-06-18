---
title: OpenID Connect Discovery 1.0
category: reference
status: stable
confidence: high
aliases: [OIDC Discovery, OpenID Connect Discovery, openid-configuration, well-known openid]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [authorization-server-discovery, rfc-8414-authorization-server-metadata, openid-connect-dynamic-client-registration, mcp-authorization, openid-foundation]
tags: [oidc, discovery, metadata, spec, openid-foundation, reference]
---

# OpenID Connect Discovery 1.0

**OpenID Connect Discovery 1.0** (from the [[openid-foundation|OpenID Foundation]]) defines the `.well-known/openid-configuration` metadata document by which an OpenID Provider advertises its endpoints and capabilities. It predates and inspired [[rfc-8414-authorization-server-metadata|RFC 8414]], with which it largely overlaps.

## Role in MCP Authorization

The second of the two AS-discovery mechanisms. Per the [[mcp-authorization-overview]], an MCP authorization server MUST provide **at least one** of [[rfc-8414-authorization-server-metadata|RFC 8414]] or OIDC Discovery, and MCP clients MUST support **both**, trying OAuth 2.0 metadata before the OIDC endpoint. See [[authorization-server-discovery]].

## Link

- Spec: <https://openid.net/specs/openid-connect-discovery-1_0.html>
