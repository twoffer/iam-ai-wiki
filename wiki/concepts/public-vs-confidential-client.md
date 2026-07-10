---
title: Public vs. Confidential Clients
category: concept
status: stable
confidence: high
aliases: ["public client", "confidential client", "OAuth client types", "client authentication", "token_endpoint_auth_method"]
enterprise_analogs: ["RFC 6749 §2.1 Client Types", "OAuth 2.1 client authentication", "RFC 7636 PKCE", "RFC 8252 OAuth for Native Apps", "private_key_jwt (RFC 7523 / OIDC client auth)", "RFC 6749 §4.4 client_credentials"]
last_updated: 2026-06-19
sources: ["mcp-authorization-overview", "mcp-authorization-client-registration", "mcp-authorization-security-considerations"]
related: ["proof-key-for-code-exchange", "oauth-2-1", "client-registration", "oauth-client-id-metadata-documents", "mcp-authorization", "token-theft", "machine-identity"]
tags: ["oauth", "client-authentication", "public-client", "confidential-client", "pkce", "mcp"]
---

# Public vs. Confidential Clients

OAuth divides clients into two types by their ability to **authenticate themselves to the authorization server** (RFC 6749 §2.1):

- **Confidential client** — can hold credentials and prove its identity at the token endpoint (a `client_secret`, or an asymmetric key via `private_key_jwt`). Typically a server-side app running where a secret can be protected.
- **Public client** — cannot keep a credential confidential, so it does **not** authenticate at the token endpoint. Typically code running on a user's device: a native/desktop app, a CLI, a single-page app, or a mobile app.

The distinction is not about trust level; it is about whether the client can possess a secret the AS can verify.

## Does MCP assume the client is always public?

**No — but public is the default case the profile is built around.** The [[mcp-authorization|MCP authorization]] profile explicitly requires authorization servers to "implement OAuth 2.1 with appropriate security measures for **both confidential and public clients**" ([[mcp-authorization-overview]], *Overview* item 1; `raw/MCPAuthorization_Overview.md:68`). A confidential path is available: a [[oauth-client-id-metadata-documents|Client ID Metadata Document]] client **MAY** authenticate with `private_key_jwt` plus a JWKS ([[mcp-authorization-client-registration]]; `raw/MCPAuthorization_ClientRegistration.md:49`).

In practice, though, MCP clients are **overwhelmingly public**. They are desktop hosts, CLIs, IDE plugins, and agent runtimes that run on the user's machine and have nowhere to hide a secret ([[proof-key-for-code-exchange]]). The canonical CIMD example in the spec is a public client — its metadata declares `"token_endpoint_auth_method": "none"` (`raw/MCPAuthorization_ClientRegistration.md:74`). So the realistic reading is: *assume public, accommodate confidential.*

## What "public client" means in practice

- **No token-endpoint authentication.** The client advertises `token_endpoint_auth_method: "none"` and sends no client secret. Its `client_id` asserts an identity but does not *prove* one.
- **Identity is claimed, not authenticated.** This is exactly why [[client-registration|client registration]] and redirect-URI validation carry more weight: with no client authentication, the AS leans on the `redirect_uri` (and, for [[oauth-client-id-metadata-documents|CIMD]], the fetched metadata document) to decide where a code may be delivered. It is also why CIMD `localhost` impersonation is a real concern — nothing cryptographic distinguishes two clients claiming the same metadata URL.
- **No client-only grants.** A public client cannot use the `client_credentials` grant (RFC 6749 §4.4), which exists to authenticate a client with no user present. Agent *workload* identity that needs `client_credentials` therefore implies a confidential client — see [[machine-identity]].
- **Compensating controls become mandatory.** Because the client can't authenticate, the profile promotes the protections that don't depend on a secret to MUSTs: [[proof-key-for-code-exchange|PKCE]] with `S256`, exact redirect-URI matching, and **refresh-token rotation** — "for public clients, authorization servers MUST rotate refresh tokens" ([[mcp-authorization-security-considerations]]; `raw/MCPAuthorization_SecurityConsiderations.md:40`; OAuth 2.1 §4.3.1). See [[token-theft]].

## Effect on the PKCE authorization-code flow

Under [[oauth-2-1|OAuth 2.1]], [[proof-key-for-code-exchange|PKCE]] is mandatory for **all** authorization-code clients, public and confidential alike, so the wire flow looks the same: `code_challenge` on the way out, `code_verifier` at redemption. The difference is **what protects the code at the token endpoint, and what else accompanies it**:

| | Confidential client | Public client (the MCP norm) |
| --- | --- | --- |
| Token-endpoint request carries | `code` + `code_verifier` **+ client authentication** (`client_secret` or `private_key_jwt`) | `code` + `code_verifier` only (`token_endpoint_auth_method: none`) |
| What binds the code to the legitimate client | **Two independent factors**: the client credential *and* PKCE | **PKCE alone** — there is no client secret to fall back on |
| If an attacker intercepts the code | Useless without *both* the client credential and the verifier | Useless without the verifier — PKCE is the *only* thing standing between the attacker and a token |
| Refresh-token handling | Rotation recommended; the authenticated client is itself a barrier | Rotation **MUST** be enforced by the AS (OAuth 2.1 §4.3.1) |

For a confidential client, PKCE is **defense in depth** layered on top of client authentication; even pre-PKCE, an intercepted code was hard to redeem because the attacker also lacked the client secret. For a public client, PKCE is **the primary defense** against authorization-code interception/injection — remove it and an intercepted code is directly redeemable by anyone, since the token endpoint asks for no other proof. This is precisely why MCP makes PKCE non-optional and additionally requires clients to **verify PKCE support from AS metadata and refuse to proceed if it is absent** (a downgrade-attack defense detailed in [[proof-key-for-code-exchange]]): the profile cannot let a public client silently lose its only code-binding protection.

## Relation to pre-AI IAM

The public/confidential split is verbatim **RFC 6749 §2.1**, and the public-client-with-PKCE pattern is the same one practitioners have shipped for native and SPA apps since **RFC 8252** and **RFC 7636**. A confidential client authenticating with `private_key_jwt` is ordinary OIDC client authentication. Nothing about the *mechanism* is new; an engineer who has built a mobile OAuth client already holds the right mental model — "no secret, lean on PKCE and redirect-URI validation."

## Why pre-AI IAM is insufficient

In classic enterprise OAuth, whether a client is public or confidential is decided **once, by a human integrator**, at registration time, and the AS knows each client individually. MCP inverts both: clients connect to authorization servers they have **no prior relationship** with ([[client-registration]]), so the AS must safely transact with an unbounded population of mostly-public clients it cannot vet ahead of time. That removes the human who, in the enterprise model, decided that *this* app could be trusted with a secret. The profile compensates by (a) making the secret-free protections mandatory rather than recommended (PKCE + `S256`, rotation, exact redirect matching, mandatory PKCE-support discovery), and (b) offering [[oauth-client-id-metadata-documents|CIMD]] as a registration-free identity that *optionally* upgrades a client to confidential via `private_key_jwt` without any pre-arranged secret exchange. The public-client default is thus a direct consequence of the open, agentic deployment model, and the hardening exists to make that default safe.
