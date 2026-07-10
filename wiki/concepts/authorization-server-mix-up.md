---
title: Authorization Server Mix-Up
category: concept
status: stable
confidence: high
aliases: ["mix-up attack", "AS mix-up", "OAuth mix-up", "issuer mix-up"]
enterprise_analogs: ["RFC 9207 Authorization Server Issuer Identification", "RFC 9700 OAuth 2.0 Security BCP", "OAuth mix-up attack"]
last_updated: 2026-06-19
sources: ["mcp-authorization-security-considerations", "mcp-authorization-server-discovery", "mcp-authorization-overview"]
related: ["rfc-9207-authorization-server-issuer-identification", "authorization-server-discovery", "rfc-8414-authorization-server-metadata", "confused-deputy", "proof-key-for-code-exchange", "security-considerations", "mcp-authorization"]
tags: ["security", "mix-up", "oauth", "threat-model", "core-concept"]
---

# Authorization Server Mix-Up

An **authorization-server mix-up** is an attack in which a client is tricked into interacting with the wrong authorization server — for example, sending an authorization code or token issued by the honest AS to an attacker-controlled endpoint, or accepting a response that originated from a different AS than the one the client believes it is talking to. It is a [[confused-deputy]] instance specific to the discovery/redirect machinery.

MCP defends at **two layers**, with two separate issuer validations:

## Defense 1 — Metadata-document issuer validation (discovery layer)

The [[mcp-authorization-server-discovery|AS Discovery]] document (*Metadata Validation*) requires that, after fetching an AS-metadata document, the client validate it per [[rfc-8414-authorization-server-metadata|RFC 8414]] §3.3 / [[openid-connect-discovery|OIDC Discovery]] §4.3: the `issuer` **inside** the document MUST be **identical** to the issuer identifier used to construct the well-known URL. A document fetched from `https://attacker.example/.well-known/oauth-authorization-server` claiming `"issuer": "https://honest.example"` MUST be rejected. This stops an attacker who controls a metadata endpoint from impersonating a trusted issuer and redirecting the client's endpoints before any redirect happens.

## Defense 2 — Authorization-response `iss` validation (redirect layer)

The [[mcp-authorization-overview]] (*Authorization Response Validation*) mandates [[rfc-9207-authorization-server-issuer-identification|RFC 9207]] issuer validation:

- Before redirecting, the client MUST **record the AS `issuer`** from validated metadata, bound to the same per-request record as the [[proof-key-for-code-exchange|PKCE]] verifier and `state`.
- On the authorization response, the client MUST compare the returned `iss` to the recorded issuer using **exact string comparison** (RFC 3986 §6.2.1) with **no normalization** (no case folding, default-port elision, trailing-slash, or percent-encoding changes).
- If the AS advertises `authorization_response_iss_parameter_supported: true` but `iss` is absent, the client MUST **reject** the response. The rule applies to error responses too — on mismatch the client MUST NOT act on or display the error fields.

The spec notes a future revision is expected to upgrade AS inclusion of `iss` from SHOULD to MUST. The two defenses are complementary: Defense 1 ensures the client fetched its endpoints from the genuine issuer; Defense 2 ensures the response it later receives came from that same issuer. Both compose with PKCE (which protects the code) and with the [[authorization-server-discovery]] rules (which constrain *which* AS is trusted).

## Why weaker measures don't suffice

The [[mcp-authorization-security-considerations|Security Considerations]] document (*Mix-Up Attacks*) is explicit that issuer identification is *necessary* — other controls do not cover this attack:

- **PKCE alone does not prevent mix-up.** In a mix-up, the client is tricked into talking to the attacker's AS and redeems the code at the attacker's token endpoint — so it hands the attacker the `code_verifier` itself. PKCE protects against an interceptor who lacks the verifier, not against a client that willingly sends it to the wrong endpoint.
- **Resource indicators do not help** when the attacker's AS intercepts requests *before* they reach the honest AS; audience binding constrains where a token is used, not which AS the client is talking to.
- **The defense depends on honest ASes emitting `iss`.** Authorization Response Validation provides no protection against an *honest* server that simply omits `iss` — there is nothing for the client to validate. This is precisely why the spec intends to make AS `iss` emission a MUST.

## Relation to pre-AI IAM

This is the textbook **OAuth mix-up attack** and its standard mitigation (RFC 9207), catalogued in the OAuth 2.0 Security BCP (RFC 9700). Practitioners who deploy multi-AS or federated OAuth already know issuer identification as the fix.

## Why pre-AI IAM is insufficient

Mix-up risk scales with **dynamic, runtime discovery of authorization servers** — exactly MCP's model, where a client connects to servers (and thus ASes) it has never seen and did not pre-configure. With no human integrator to pin the AS in advance, the client must validate issuer identity automatically on every flow, which is why MCP makes RFC 9207 validation mandatory.
