---
title: MCP Authorization — Security Considerations
category: topic
status: stable
confidence: high
aliases: [MCP security considerations, MCP authorization security]
enterprise_analogs: [OAuth 2.1 §7 Security Considerations, RFC 9700 OAuth 2.0 Security BCP, RFC 6819 OAuth threat model, RFC 9207 issuer identification, RFC 8707 audience restriction, RFC 9068 JWT access tokens]
last_updated: 2026-07-08
sources: [mcp-authorization-security-considerations, mcp-authorization-overview, mcp-security-best-practices]
related: [mcp-authorization, token-audience-binding, token-passthrough, confused-deputy, authorization-server-mix-up, proof-key-for-code-exchange, open-redirection, token-theft, server-side-request-forgery, oauth-client-id-metadata-documents, canonical-server-uri, oauth-2-1, mcp-security-best-practices, session-hijacking, local-mcp-server-security, authorization-url-injection, prompt-injection]
tags: [security, threat-model, mcp]
---

# MCP Authorization — Security Considerations

The fourth and final document of the MCP Authorization specification defines the **normative security requirements** every implementation MUST follow ([[mcp-authorization-security-considerations]]). The other three documents — the [[mcp-authorization-overview|overview]], [[authorization-server-discovery|AS Discovery]], and [[client-registration]] — defer their security rationale here. It defines no new mechanism; it is an OAuth threat-model checklist re-stated as MCP MUSTs, layered on top of [[oauth-2-1|OAuth 2.1]] §7, which implementations MUST follow in full.

This page synthesizes the document's nine threat areas. For the per-threat concept pages, follow the links.

## Token audience binding and validation

[[rfc-8707-resource-indicators|RFC 8707]] Resource Indicators bind a token to its intended audience *when the AS supports the capability*. Two MUSTs make this work end to end:

- MCP clients **MUST** include the `resource` parameter ([[canonical-server-uri|canonical URI]]) in authorization **and** token requests.
- MCP servers **MUST** validate that presented tokens were issued specifically for them.

Token passthrough is "explicitly forbidden." This is the structural defense against the [[confused-deputy]] problem; see [[token-audience-binding]] and *Access token privilege restriction* below.

## Token theft

Tokens stored by the client or cached/logged on the server let an attacker make requests that look legitimate. Mitigations:

- Clients and servers **MUST** implement secure token storage and follow OAuth best practices ([[oauth-2-1|OAuth 2.1]] §7.1).
- ASes **SHOULD** issue **short-lived** access tokens to limit the blast radius of a leak.
- For **public clients** (the typical MCP client), ASes **MUST** rotate refresh tokens ([[oauth-2-1|OAuth 2.1]] §4.3.1).

See [[token-theft]].

## Communication security

Implementations **MUST** follow [[oauth-2-1|OAuth 2.1]] §1.5. Concretely: every AS endpoint **MUST** be served over HTTPS, and every redirect URI **MUST** be either `localhost` or HTTPS.

## Authorization code protection (PKCE and PKCE discovery)

An attacker who intercepts an authorization code may try to redeem it. MCP clients **MUST** implement [[proof-key-for-code-exchange|PKCE]] and **MUST** use the `S256` challenge method when technically capable. The document adds a requirement the [[mcp-authorization-overview|overview]] only implied — clients **MUST verify PKCE support before proceeding**, using AS metadata as the discovery channel (PKCE itself defines no discovery mechanism):

| Discovery source | Rule |
| --- | --- |
| [[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata]] | If `code_challenge_methods_supported` is **absent**, the AS does not support PKCE and the client **MUST** refuse to proceed. |
| [[openid-connect-discovery|OIDC Discovery 1.0]] | The field is not defined by OIDC metadata but is commonly present; clients **MUST** verify its presence and **MUST** refuse to proceed if absent. ASes offering OIDC Discovery **MUST** include it for MCP compatibility. |

Refusing to proceed when PKCE support is unverifiable is a downgrade-attack defense: it stops an attacker (or a misconfigured AS) from silently dropping the protection that defeats code injection.

## Mix-up attacks

When a client talks to many ASes, an attacker controlling one may try to have the client send it a code or token issued by a *different, honest* AS ([RFC 9207 §1](https://datatracker.ietf.org/doc/html/rfc9207#section-1)). The [[mcp-authorization-overview|overview]]'s *Authorization Response Validation* binds the response to the AS the client recorded before redirecting. This document sharpens **why weaker measures are not enough**:

- **PKCE alone does not prevent mix-up** — the client transmits the `code_verifier` to the attacker's token endpoint, so the attacker obtains what it needs.
- **Resource indicators do not help** when the attacker's AS intercepts requests before they reach the honest AS.
- The mitigation **depends on honest ASes emitting `iss`**; it provides no protection against an honest server that omits it (which is why the spec intends to upgrade AS `iss` emission from SHOULD to MUST).

See [[authorization-server-mix-up]] and [[rfc-9207-authorization-server-issuer-identification]].

## Open redirection

An attacker may craft malicious redirect URIs to send users to phishing sites. Requirements:

- MCP clients **MUST** register redirect URIs with the AS.
- ASes **MUST** validate **exact** redirect URIs against pre-registered values.
- Clients **SHOULD** use and verify `state` parameters, discarding results that are missing or mismatched.
- ASes **MUST** take precautions against redirecting user agents to untrusted URIs ([[oauth-2-1|OAuth 2.1]] §7.12.2) and **SHOULD** only auto-redirect to trusted URIs — otherwise informing the user and relying on their decision.

See [[open-redirection]].

## Client ID Metadata Document security

[[oauth-client-id-metadata-documents|Client ID Metadata Documents (CIMD)]] let a client present an HTTPS URL as its `client_id` that the AS fetches on demand. That fetch and the open client population create three sub-threats ([draft §6](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00#name-security-considerations)):

- **Authorization-server abuse / [[server-side-request-forgery|SSRF]].** The AS fetches an attacker-supplied URL; without protection it can be steered into requesting arbitrary internal endpoints. ASes fetching metadata documents **SHOULD** consider SSRF risks.
- **`localhost` redirect-URI impersonation.** CIMD cannot by itself stop an attacker from presenting a legitimate client's metadata URL as their `client_id`, binding to any `localhost` port, and supplying it as the `redirect_uri` to capture the code — while the user sees the legitimate client's name. ASes **SHOULD** warn on `localhost`-only redirect URIs, **MAY** require additional attestation, and **MUST** clearly display the redirect URI hostname during authorization.
- **Trust policies.** ASes **MAY** apply domain-based policies — allowlists, accept-any-HTTPS for open servers, reputation checks, domain-age/certificate restrictions, and prominent display of the CIMD and associated hostnames to resist phishing. Servers retain full control over access policy.

See [[client-registration]] for how CIMD fits the registration model.

## Confused deputy problem

An MCP server that acts as an intermediary to third-party APIs is a confused deputy by construction. Using stolen authorization codes, an attacker can obtain tokens without user consent. The specific normative rule:

> MCP **proxy servers using static client IDs MUST obtain user consent for each [[client-registration|dynamically registered client]]** before forwarding to third-party authorization servers (which may require additional consent).

The hazard is that a static, pre-approved upstream `client_id` lets the third-party AS skip its consent screen (the user already consented once), so a malicious dynamically-registered downstream client can ride that prior consent. See [[confused-deputy]] and the [[mcp-security-best-practices|Security Best Practices]] guide.

## Access token privilege restriction

The capstone requirement, with two failure modes:

1. **Audience validation failures.** A server that does not verify the audience claim ([[rfc-9068-jwt-access-tokens|RFC 9068]]) may accept tokens issued for other services, breaking an OAuth security boundary and enabling cross-service token reuse.
2. **Token passthrough.** Forwarding such tokens unmodified to downstream services produces the confused-deputy problem.

Therefore MCP servers **MUST** validate inbound tokens ([[oauth-2-1|OAuth 2.1]] §5.2) before processing, **MUST** accept only tokens that name them in the audience claim, and **MUST** reject the rest. If a server calls upstream APIs it acts as an OAuth **client** to them and uses a **separate** token issued by the upstream AS — it **MUST NOT** pass through the token received from the MCP client. Clients **MUST** implement the `resource` parameter ([[rfc-8707-resource-indicators|RFC 8707]]), aligning with [RFC 9728 §7.4](https://datatracker.ietf.org/doc/html/rfc9728#section-7.4). See [[token-passthrough]] and [[token-audience-binding]].

## The companion Security Best Practices guide

The separate (non-spec) [[mcp-security-best-practices|Security Best Practices]] guide — which this document defers to for the token-passthrough and confused-deputy rationale — extends the normative checklist above with attack classes that sit outside the OAuth flow proper: the full consent-cookie anatomy and proxy mitigation stack for the [[confused-deputy]] case, the four risk classes behind the [[token-passthrough]] prohibition, [[server-side-request-forgery|SSRF]] against the *client's* discovery fetches (the mirror of the CIMD case above), [[session-hijacking]] (including its prompt-injection variant), [[local-mcp-server-security|local MCP server compromise]], [[authorization-url-injection|malicious authorization URLs]] (XSS/RCE), and a [[scope-selection-strategy|scope-minimization]] model. Implementers should treat the two documents as one threat model split across a normative and an advisory tier.

## Relation to pre-AI IAM

These map almost one-to-one onto the **OAuth 2.0 Security Best Current Practice (RFC 9700)** and the original threat model (RFC 6819): audience restriction, short-lived and rotated tokens, TLS everywhere, PKCE with `S256`, issuer identification, exact redirect-URI matching, and SSRF defenses for server-side fetches. A practitioner's OAuth threat model transfers directly; the MCP document's job is to tighten *which* mitigations are mandatory and to add a few MCP-specific items (PKCE-support discovery via metadata, CIMD `localhost` impersonation, consent-per-dynamic-client for proxy servers).

## Why pre-AI IAM is insufficient

The agentic deployment model — open client/server populations discovered at runtime, plus agents steerable by untrusted input — raises both the likelihood and the impact of these classic threats, which is why MCP elevates several optional OAuth hardening measures to requirements:

- With **no human integrator** to pin the trusted AS or pre-register the client, mix-up and confused-deputy exposure is routine rather than exceptional, and issuer/audience validation must run automatically on every flow.
- **Registration-free client identity (CIMD)** adds a *new* server-side attack surface absent from pre-registration: the AS fetches attacker-supplied URLs ([[server-side-request-forgery|SSRF]]) and must contend with `localhost` redirect impersonation.
- **MCP proxy/intermediary servers** are confused deputies by construction; consent-per-dynamic-client is a requirement with no clean pre-AI analog.

The new agentic wrinkle the document does **not** itself address is **[[prompt-injection|prompt injection as an authorization-bypass vector]]** — untrusted content steering the deputy. The [[mcp-security-best-practices|Security Best Practices]] guide supplies the corpus's first concrete instance ([[session-hijacking|session hijack prompt injection]]); the broader analysis lives on [[confused-deputy]] and [[prompt-injection]] rather than in this normative checklist.
