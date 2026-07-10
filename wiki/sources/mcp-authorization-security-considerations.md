---
title: MCP Authorization Specification — Security Considerations
category: source
status: stable
confidence: high
aliases: ["MCP Security Considerations", "MCP authorization security spec", "MCP auth security considerations"]
enterprise_analogs: ["OAuth 2.1 §7 Security Considerations", "RFC 9700 OAuth 2.0 Security BCP", "RFC 6819 OAuth 2.0 threat model", "RFC 9207 issuer identification", "RFC 8707 audience restriction", "RFC 9068 JWT access tokens"]
last_updated: 2026-06-19
sources: ["raw/MCPAuthorization_SecurityConsiderations.md"]
related: ["mcp-authorization", "mcp-authorization-overview", "security-considerations", "token-audience-binding", "token-passthrough", "confused-deputy", "authorization-server-mix-up", "proof-key-for-code-exchange", "open-redirection", "token-theft", "server-side-request-forgery", "oauth-client-id-metadata-documents", "oauth-2-1", "mcp-security-best-practices"]
tags: ["mcp", "oauth", "security", "threat-model", "spec", "source"]
---

# MCP Authorization Specification — Security Considerations

Summary of `raw/MCPAuthorization_SecurityConsiderations.md`, pulled 2026-06-17 from <https://modelcontextprotocol.io/specification/draft/basic/authorization/security-considerations> (the `draft` revision of the Model Context Protocol). This is the fourth and final document of the MCP Authorization specification; the others are the [[mcp-authorization-overview|overview]], [[authorization-server-discovery|Authorization Server Discovery]], and [[client-registration|Client Registration]]. See the topic page [[security-considerations]] for the synthesized threat model.

## What the document is

It is the **normative security requirements** the rest of the MCP Authorization spec defers to. It defines no new protocol; it enumerates threats and the MUST/SHOULD mitigations implementers must apply, layering them on top of [OAuth 2.1 §7 "Security Considerations"](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13#name-security-considerations), which it requires implementers to follow in full. It repeatedly cross-references the separate (non-spec) [[mcp-security-best-practices|Security Best Practices]] guide for the [[token-passthrough]] and [[confused-deputy]] discussions.

## Key normative claims

### Token audience binding and validation

[[rfc-8707-resource-indicators|RFC 8707]] Resource Indicators bind tokens to their intended audiences **when the AS supports the capability**. MCP clients **MUST** include the `resource` parameter in authorization and token requests; MCP servers **MUST** validate that tokens presented to them were specifically issued for their use. Token passthrough is "explicitly forbidden." See [[token-audience-binding]], [[canonical-server-uri]].

### Token theft

Attackers who obtain tokens stored by the client, or cached/logged on the server, can make requests that appear legitimate. Clients and servers **MUST** implement secure token storage and follow OAuth best practices ([OAuth 2.1 §7.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13#section-7.1)). Authorization servers **SHOULD** issue short-lived access tokens to reduce the impact of leaks. For public clients, ASes **MUST** rotate refresh tokens ([OAuth 2.1 §4.3.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13#section-4.3.1)). See [[token-theft]].

### Communication security

Implementations **MUST** follow [OAuth 2.1 §1.5](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-13#section-1.5). Specifically: (1) all AS endpoints **MUST** be served over HTTPS; (2) all redirect URIs **MUST** be either `localhost` or use HTTPS.

### Authorization code protection (PKCE + discovery)

An attacker with an intercepted authorization code can try to redeem it (OAuth 2.1 §7.5). MCP clients **MUST** implement [[proof-key-for-code-exchange|PKCE]] (OAuth 2.1 §7.5.2) **and MUST verify PKCE support before proceeding**, and **MUST** use the `S256` challenge method when technically capable (OAuth 2.1 §4.1.1). Because neither OAuth 2.1 nor PKCE defines a discovery mechanism for PKCE support, clients **MUST** rely on AS metadata:

- **[[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata]]:** if `code_challenge_methods_supported` is absent, the AS does not support PKCE and the client **MUST** refuse to proceed.
- **[[openid-connect-discovery|OIDC Discovery 1.0]]:** the OpenID Provider Metadata does not define `code_challenge_methods_supported`, but it is commonly included; clients **MUST** verify its presence and **MUST** refuse to proceed if absent. ASes providing OIDC Discovery **MUST** include `code_challenge_methods_supported` for MCP compatibility.

### Mix-up attacks

A client interacts with many ASes over its lifetime; an attacker controlling one may try to have the client send it a code/token issued by a different, honest AS ([RFC 9207 §1](https://datatracker.ietf.org/doc/html/rfc9207#section-1)). *Authorization Response Validation* (in the overview) mitigates this by binding the response to the AS the client recorded before redirecting. The document adds three sharp clarifications: **PKCE alone does not prevent mix-up** (the client transmits the `code_verifier` to the attacker's token endpoint); **resource indicators do not help** when the attacker's AS intercepts requests before they reach the honest AS; and the mitigation **depends on honest ASes emitting `iss`** — it gives no protection against an honest server that omits it. See [[authorization-server-mix-up]].

### Open redirection

An attacker may craft malicious redirect URIs to direct users to phishing sites. MCP clients **MUST** have redirect URIs registered with the AS. ASes **MUST** validate exact redirect URIs against pre-registered values. Clients **SHOULD** use and verify `state` parameters and discard results that are missing or mismatched. ASes **MUST** take precautions against redirecting user agents to untrusted URIs (OAuth 2.1 §7.12.2) and **SHOULD** only auto-redirect to trusted redirection URIs — otherwise informing the user and relying on their decision. See [[open-redirection]].

### Client ID Metadata Document security

When implementing [[oauth-client-id-metadata-documents|Client ID Metadata Documents]], ASes **MUST** consider the security implications in [draft §6](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00#name-security-considerations):

- **Authorization Server Abuse Protection ([[server-side-request-forgery|SSRF]]).** The AS takes a URL from an unknown client and fetches it; a malicious client could use this to make the AS issue requests to arbitrary URLs (e.g., private admin endpoints). ASes fetching metadata documents **SHOULD** consider SSRF risks.
- **Localhost redirect URI risks.** CIMD cannot by itself prevent `localhost` URL impersonation: an attacker can present a legitimate client's metadata URL as their `client_id`, bind to any `localhost` port and supply it as the `redirect_uri`, and receive the code when the user approves — while the AS shows the legitimate client's name. ASes **SHOULD** display extra warnings for `localhost`-only redirect URIs, **MAY** require additional attestation, and **MUST** clearly display the redirect URI hostname during authorization.
- **Trust policies.** ASes **MAY** implement domain-based trust policies: allowlists for trusted domains, accept-any-HTTPS-`client_id` for open servers, reputation checks, restrictions by domain age or certificate validation, and prominent display of the CIMD and associated client hostnames to prevent phishing. Servers retain full control over their access policies.

### Confused deputy problem

Attackers can exploit MCP servers acting as intermediaries to third-party APIs; using stolen authorization codes they can obtain access tokens without user consent. MCP proxy servers using **static client IDs MUST obtain user consent for each [[client-registration|dynamically registered client]]** before forwarding to third-party authorization servers (which may require additional consent). See [[confused-deputy]] and the [[mcp-security-best-practices|Security Best Practices]] guide.

### Access token privilege restriction

An attacker can compromise an MCP server that accepts tokens issued for other resources. Two dimensions:

1. **Audience validation failures.** Not verifying that tokens were intended for it (e.g., via the audience claim per [[rfc-9068-jwt-access-tokens|RFC 9068]]) lets a server accept tokens issued for other services, breaking an OAuth security boundary.
2. **Token passthrough.** Forwarding such tokens unmodified to downstream services causes the [[confused-deputy]] problem.

Servers **MUST** validate access tokens before processing (OAuth 2.1 §5.2), ensuring the token was issued specifically for them, and **MUST** reject tokens that do not include them in the audience claim. If the server calls upstream APIs it acts as an OAuth client to them, using a **separate** token issued by the upstream AS; it **MUST NOT** pass through the token received from the MCP client. Clients **MUST** implement the `resource` parameter ([[rfc-8707-resource-indicators|RFC 8707]]), aligning with [RFC 9728 §7.4](https://datatracker.ietf.org/doc/html/rfc9728#section-7.4). See [[token-passthrough]], [[token-audience-binding]].

## Relation to pre-AI IAM

The document is, essentially, an **OAuth threat-model checklist** re-stated as MCP MUSTs. Every item has a direct ancestor in the OAuth 2.0 Security Best Current Practice ([RFC 9700](https://datatracker.ietf.org/doc/html/rfc9700)) and the original threat model (RFC 6819): audience restriction (RFC 8707), short-lived/rotated tokens, TLS everywhere, PKCE with `S256`, issuer identification (RFC 9207), exact redirect-URI matching, and SSRF defenses for server-side fetches. A practitioner's OAuth threat model transfers almost verbatim; the document's contribution is to mark *which* mitigations are mandatory for MCP and to add a small number of MCP-specific wrinkles (PKCE-support discovery via metadata, CIMD `localhost` impersonation, consent-per-dynamic-client for proxy servers).

## Why pre-AI IAM is insufficient

The agentic deployment model raises both the likelihood and the impact of these classic threats, which is why MCP elevates several optional OAuth hardening measures to requirements:

- **Open, runtime-discovered AS/client populations** make mix-up and confused-deputy exposure routine rather than exceptional — there is no human integrator to pin the trusted AS or pre-register the client.
- **Registration-free client identity (CIMD)** introduces a *new* server-side attack surface: the AS now fetches attacker-supplied URLs, importing classic [[server-side-request-forgery|SSRF]] into the OAuth control plane, and `localhost` redirect impersonation that pre-registration did not permit.
- **MCP proxy/intermediary servers** brokering third-party APIs are confused deputies by construction; consent-per-dynamic-client is the new requirement that has no clean pre-AI analog.

The document does not itself treat **prompt injection as an authorization-bypass vector** — that broader agentic concern lives in the [[confused-deputy]] analysis and the [[mcp-security-best-practices|Security Best Practices]] guide rather than here.
