---
title: Canonical Server URI & the Resource Parameter
category: topic
status: stable
confidence: high
aliases: [canonical server URI, resource parameter, resource indicator, canonical MCP URI]
enterprise_analogs: [RFC 8707 Resource Indicators, RFC 9728 resource identifier, RFC 3986 URI normalization]
last_updated: 2026-06-19
sources: [mcp-authorization-overview, mcp-authorization-security-considerations]
related: [token-audience-binding, rfc-8707-resource-indicators, rfc-9728-protected-resource-metadata, mcp-authorization, security-considerations]
tags: [oauth, resource-indicators, audience, uri, mcp]
---

# Canonical Server URI & the Resource Parameter

To bind a token to its intended audience ([[token-audience-binding]]), an [[mcp-authorization|MCP client]] must name the target server unambiguously. This page covers MCP's use of the [[rfc-8707-resource-indicators|RFC 8707]] `resource` parameter and the rules for the **canonical URI** that identifies an MCP server ([[mcp-authorization-overview]], *Resource Parameter Implementation*).

## The resource parameter

MCP clients **MUST** implement RFC 8707. The `resource` parameter:

1. **MUST** be included in **both** the authorization request and the token request.
2. **MUST** identify the MCP server the token will be used with.
3. **MUST** use the server's **canonical URI** (RFC 8707 §2), aligned with the `resource` of [[rfc-9728-protected-resource-metadata|RFC 9728]].

Clients **MUST** send it **regardless of whether the AS supports it**. The [[mcp-authorization-security-considerations|Security Considerations]] document (*Access Token Privilege Restriction*) reaffirms this as a MUST and pins it to [RFC 9728 §7.4](https://datatracker.ietf.org/doc/html/rfc9728#section-7.4) — the canonical URI is the client-side input that lets the AS bind the token's audience. Example (URL-encoded):

```
&resource=https%3A%2F%2Fmcp.example.com
```

## Canonical URI rules

The canonical URI is the resource identifier from RFC 8707 §2. Clients SHOULD provide the **most specific** URI they can for the target server. Canonical form uses lowercase scheme and host, but implementations SHOULD accept uppercase scheme/host for robustness.

**Valid examples:**

- `https://mcp.example.com/mcp`
- `https://mcp.example.com`
- `https://mcp.example.com:8443`
- `https://mcp.example.com/server/mcp` (path needed to identify an individual server)

**Invalid examples:**

- `mcp.example.com` (missing scheme)
- `https://mcp.example.com#fragment` (contains a fragment)

Both trailing-slash and non-trailing-slash forms are technically valid (RFC 3986), but implementations SHOULD consistently use the **no-trailing-slash** form unless the slash is semantically significant. (Note the contrast with [[rfc-9207-authorization-server-issuer-identification|`iss` comparison]], which forbids *any* normalization — there the comparison is exact-string, whereas here the spec gives interoperability latitude on scheme/host case.)

## Relation to pre-AI IAM

This is RFC 8707 Resource Indicators used exactly as designed: name the resource so the AS can mint an audience-restricted token. Enterprises already pass `resource`/`audience` to keep multi-API tokens from being over-broad. The canonical-URI discipline mirrors the care any practitioner takes with issuer/audience string identity.

## Why pre-AI IAM is insufficient

Because an MCP client targets **many independently operated servers discovered at runtime**, the `resource` parameter is promoted from optional to **mandatory and unconditional** — sent even when the AS ignores it — so that audience binding is never silently dropped. In static enterprise OAuth, a forgotten audience parameter is often harmless because a human pre-wired which APIs trust which tokens; in the open agentic setting it is a [[confused-deputy]] vulnerability, so the spec leaves no room to omit it.
