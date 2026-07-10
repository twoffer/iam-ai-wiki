---
title: RFC 6750 — OAuth 2.0 Bearer Token Usage
category: reference
status: stable
confidence: high
aliases: ["RFC 6750", "Bearer Token Usage", "WWW-Authenticate Bearer", "insufficient_scope"]
enterprise_analogs: []
last_updated: 2026-06-18
sources: ["mcp-authorization-overview"]
related: ["mcp-authorization", "scope-selection-strategy", "step-up-authorization", "oauth-2-1"]
tags: ["oauth", "bearer-token", "spec", "ietf", "reference"]
---

# RFC 6750 — OAuth 2.0 Bearer Token Usage

**RFC 6750** defines how bearer tokens are presented to a resource server and how the server challenges for them. It specifies the `Authorization: Bearer <token>` header, the `WWW-Authenticate: Bearer` challenge, and the standard error codes — including `insufficient_scope` and the optional `scope` parameter that tells a client which scopes a resource requires.

## Role in MCP Authorization

Used in several places by the [[mcp-authorization-overview]]:

- **§3** — the `scope` parameter in `WWW-Authenticate` that drives the [[scope-selection-strategy]].
- **§3.1** — `insufficient_scope` with `403 Forbidden`, the basis of [[step-up-authorization]] runtime scope challenges.
- Bearer presentation of access tokens on every MCP request (the header form; tokens never in the query string).

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc6750>
