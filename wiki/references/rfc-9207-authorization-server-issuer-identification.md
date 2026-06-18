---
title: RFC 9207 — OAuth 2.0 Authorization Server Issuer Identification
category: reference
status: stable
confidence: high
aliases: [RFC 9207, Issuer Identification, iss parameter, authorization_response_iss_parameter_supported]
enterprise_analogs: []
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [authorization-server-mix-up, mcp-authorization, authorization-server-discovery, proof-key-for-code-exchange, confused-deputy]
tags: [oauth, security, mix-up, spec, ietf, reference]
---

# RFC 9207 — OAuth 2.0 Authorization Server Issuer Identification

**RFC 9207** adds an `iss` parameter to the authorization response so a client can confirm *which* authorization server issued the response, defeating [[authorization-server-mix-up|mix-up attacks]]. An AS that emits `iss` advertises `authorization_response_iss_parameter_supported: true` in its metadata (§2.3). Clients validate the returned `iss` against the expected issuer (§2.4).

## Role in MCP Authorization

The [[mcp-authorization-overview]] (*Authorization Response Validation*) makes `iss` validation a client **MUST**, with AS emission a **SHOULD** today (expected to become MUST in a future revision). Specific rules:

- Record the AS `issuer` (from validated metadata) before redirecting, bound to the PKCE/`state` record.
- Compare a returned `iss` by **exact string comparison** (RFC 3986 §6.2.1) with **no normalization**.
- If the AS advertises support but `iss` is absent, **reject**. Applies to error responses too.

The overview also applies a local-policy provision: a present `iss` is validated even if metadata doesn't advertise support.

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc9207>
