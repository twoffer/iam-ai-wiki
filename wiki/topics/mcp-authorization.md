---
title: MCP Authorization
category: topic
status: evolving
confidence: high
aliases: [Model Context Protocol authorization, MCP auth, MCP OAuth profile]
enterprise_analogs: [OAuth 2.1 draft-ietf-oauth-v2-1-13, RFC 6749 §1.1 roles, RFC 6750 Bearer Token Usage, RFC 8707 Resource Indicators, RFC 9728 Protected Resource Metadata]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-specification, delegated-authorization, authorization-server-discovery, client-registration, scope-selection-strategy, canonical-server-uri, token-audience-binding, proof-key-for-code-exchange, step-up-authorization, token-passthrough, security-considerations, oauth-2-1]
tags: [mcp, oauth, authorization, profile]
---

# MCP Authorization

**MCP Authorization** is the transport-level authorization model for the [[mcp-specification|Model Context Protocol]]. It lets an MCP client make requests to a restricted MCP server on behalf of a resource owner, using OAuth tokens. It is defined as a **profile of [[oauth-2-1|OAuth 2.1]]** — a curated subset of existing OAuth/OIDC specifications, not a new protocol ([[mcp-authorization-overview]]).

Authorization is **OPTIONAL** in MCP. The profile targets **HTTP-based transports**, which SHOULD conform to it. **STDIO transports SHOULD NOT** use it and instead read credentials from the environment (see [[machine-identity]]); other transports MUST follow their own security best practices ([[mcp-authorization-overview]]).

## Roles

MCP reuses the standard OAuth three-party model verbatim:

| MCP term | OAuth 2.1 role | Responsibility |
| --- | --- | --- |
| MCP server | Resource server | Accepts and validates access tokens; serves protected MCP requests |
| MCP client | Client | Obtains tokens and makes protected requests on behalf of a resource owner |
| Authorization server (AS) | Authorization server | Authenticates the user (if needed) and issues access tokens |

The AS may be co-located with the MCP server or run as a separate entity; its implementation is out of scope for the spec. [[authorization-server-discovery]] defines how a server points clients at its AS ([[mcp-authorization-overview]]).

## Mandatory building blocks

The profile makes several otherwise-optional OAuth features **mandatory**:

- **Authorization servers MUST implement [[oauth-2-1|OAuth 2.1]]** for both confidential and public clients.
- **MCP servers MUST implement [[rfc-9728-protected-resource-metadata|Protected Resource Metadata]] (RFC 9728)**, and clients MUST use it to discover the AS. See [[authorization-server-discovery]].
- **Authorization servers MUST provide at least one of** [[rfc-8414-authorization-server-metadata|AS Metadata (RFC 8414)]] or [[openid-connect-discovery|OIDC Discovery]]; **clients MUST support both**.
- **Clients MUST implement [[rfc-8707-resource-indicators|Resource Indicators (RFC 8707)]]** — every authorization and token request carries a `resource` parameter naming the target server's [[canonical-server-uri|canonical URI]].
- **Servers MUST validate token audience** — a token is accepted only if it was issued for that server ([[token-audience-binding]]).
- **Clients MUST use [[proof-key-for-code-exchange|PKCE]]** and MUST validate the authorization-response `iss` ([[rfc-9207-authorization-server-issuer-identification|RFC 9207]]) before redeeming a code.

Client identity is established through one of three [[client-registration]] mechanisms (Client ID Metadata Documents, pre-registration, or deprecated Dynamic Client Registration). Scopes are negotiated via the [[scope-selection-strategy]] and escalated via [[step-up-authorization]].

## The authorization flow

The complete flow (*Authorization Flow Steps* in [[mcp-authorization-overview]]):

1. Client makes an MCP request **without a token**.
2. Server returns **`401 Unauthorized`** with a `WWW-Authenticate` header carrying `resource_metadata` (and SHOULD-include `scope`).
3. Client fetches the server's **Protected Resource Metadata** and extracts the authorization server(s).
4. Client performs **[[authorization-server-discovery|AS metadata discovery]]** (OAuth 2.0 then OIDC endpoints, in priority order) and records the AS `issuer`.
5. Client obtains a `client_id` via one of the [[client-registration]] mechanisms.
6. Client generates **PKCE** parameters, includes the **`resource`** parameter, applies the **scope selection strategy**, and opens the browser to the authorization endpoint.
7. User authorizes; the AS redirects back with an **authorization code** and **`iss`**.
8. Client **validates `iss`** against the recorded issuer (exact string comparison, no normalization), then exchanges the code at the token endpoint with `code_verifier` + `resource`.
9. AS returns an **access token** (and optionally a **refresh token**).
10. Client sends `Authorization: Bearer <token>` on every subsequent MCP request.

## Token usage and handling

Access tokens MUST be sent in the `Authorization: Bearer` header on **every** request and MUST NOT appear in the query string. Servers validate tokens per [[oauth-2-1|OAuth 2.1]] §5.2 and MUST confirm the token's audience is themselves ([[token-audience-binding]]). Invalid/expired tokens get `401`. Clients MUST NOT send a server any token other than one issued by that server's AS, and servers MUST NOT accept or transit foreign tokens — the [[token-passthrough]] prohibition ([[mcp-authorization-overview]]).

## Refresh tokens

Clients that want refresh tokens keep them confidential, SHOULD list `refresh_token` in `grant_types`, and MAY request the `offline_access` scope when the AS advertises it. The AS retains discretion to issue them or not. MCP servers (resource servers) SHOULD NOT surface `offline_access` in `WWW-Authenticate` or `scopes_supported`, since refresh is not a resource requirement ([[mcp-authorization-overview]]).

## Error handling

| Status | Meaning |
| --- | --- |
| `401 Unauthorized` | Authorization required, or token invalid/expired |
| `403 Forbidden` | Insufficient scope or permission (triggers [[step-up-authorization]]) |
| `400 Bad Request` | Malformed authorization request |

Runtime insufficient-scope is signalled with `403` + `WWW-Authenticate: Bearer error="insufficient_scope"` plus a `scope` challenge and `resource_metadata`. See [[scope-selection-strategy]] and [[step-up-authorization]].

## Relation to pre-AI IAM

This is, deliberately, ordinary OAuth 2.1. A practitioner's mental model of the **authorization code grant with PKCE for a public client** ([[oauth-2-1]], formerly RFC 6749 §4.1 + RFC 7636) carries over unchanged: the MCP server is a resource server (RFC 6749 §1.1), the bearer token is presented per RFC 6750, the AS is discovered the way any RFC 8414 / OIDC Discovery client discovers one, and audience restriction is the RFC 8707 `resource` parameter that enterprises already use for multi-API token scoping. There is no novel grant type and no novel credential. The contribution is *constraining* the option space so independently built clients and servers interoperate securely without prior coordination — the same role that a tightly written FAPI profile plays for open banking.

## Why pre-AI IAM is insufficient

Classic OAuth deployments assume a **known, pre-registered client** talking to a **known set of resource servers**, integrated by humans ahead of time. MCP breaks both assumptions:

- **Open client/server population.** An MCP client (e.g., a chat host or agent runtime) discovers and connects to servers it has never seen, operated by parties it has no registration relationship with. This forces first-class **discovery** (RFC 9728 mandatory on the server) and **registration-free client identity** ([[client-registration|Client ID Metadata Documents]]) — features that are optional or skipped in traditional enterprise OAuth.
- **The client is general-purpose and not domain-aware.** A human-integrated app knows exactly which scopes it needs; a general MCP client does not, so the profile pushes scope decisions to runtime challenges and a [[scope-selection-strategy|least-privilege selection strategy]] with [[step-up-authorization]].
- **Confused-deputy exposure is acute.** Because one agent brokers many tools/servers and can be steered by untrusted input, audience-bound tokens become **mandatory** (not advisory) to keep a token meant for server A from being replayed against server B. See [[token-audience-binding]], [[token-passthrough]], and [[confused-deputy]].

In short, MCP authorization is standard OAuth with the *optional-but-recommended* security controls promoted to *required*, precisely because the agentic deployment model removes the human integrator who used to enforce them.

## Sub-pages

- [[authorization-server-discovery]] — how clients find and validate the AS (RFC 9728 → RFC 8414 / OIDC).
- [[client-registration]] — Client ID Metadata Documents, pre-registration, Dynamic Client Registration.
- [[scope-selection-strategy]] — least-privilege scope requests and `WWW-Authenticate` guidance.
- [[canonical-server-uri]] — the `resource` parameter and canonical URI rules (RFC 8707).
- [[security-considerations]] — normative threat mitigations (forthcoming ingest).
