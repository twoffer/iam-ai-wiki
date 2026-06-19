---
title: Proof Key for Code Exchange (PKCE)
category: concept
status: stable
confidence: high
aliases: [PKCE, code_challenge, code_verifier, RFC 7636]
enterprise_analogs: [RFC 7636 PKCE, OAuth 2.1 §4.1.1 (mandatory PKCE + S256), OAuth 2.1 §7.5.2 code interception, authorization code injection defense]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations, mcp-authorization-overview]
related: [mcp-authorization, oauth-2-1, rfc-7636-pkce, rfc-8414-authorization-server-metadata, openid-connect-discovery, authorization-server-mix-up, confused-deputy, security-considerations]
tags: [oauth, pkce, security, core-concept]
---

# Proof Key for Code Exchange (PKCE)

**PKCE** binds an OAuth authorization code to the specific client instance that requested it, defeating authorization-code interception/injection. The client generates a secret `code_verifier`, sends its hash as `code_challenge` in the authorization request, and proves possession by sending the `code_verifier` at the token endpoint. An intercepted code is worthless without the verifier.

In MCP it is **not optional**: the [[mcp-authorization|MCP authorization]] flow has the client "Generate PKCE parameters" as a required step ([[mcp-authorization-overview]], *Authorization Flow Steps*), and [[oauth-2-1|OAuth 2.1]] mandates PKCE for all authorization-code clients (public and confidential). This matters because MCP clients are typically **public clients** (desktop hosts, CLIs, agent runtimes) that cannot keep a client secret.

## Role in the MCP flow

PKCE is generated alongside the [[canonical-server-uri|`resource` parameter]] and the recorded `issuer` before the browser redirect, then the `code_verifier` accompanies the code at token exchange. It composes with — but is distinct from — the [[rfc-9207-authorization-server-issuer-identification|`iss` validation]] that defends [[authorization-server-mix-up]]: PKCE protects the *code*, `iss` validation protects against talking to the *wrong AS*. (PKCE notably does **not** defend mix-up on its own — see [[authorization-server-mix-up|why weaker measures don't suffice]].)

## Mandatory PKCE-support discovery

The [[mcp-authorization-security-considerations|Security Considerations]] document (*Authorization Code Protection*) adds two requirements beyond "use PKCE":

- **Use `S256`.** Clients MUST use the `S256` code-challenge method when technically capable ([[oauth-2-1|OAuth 2.1]] §4.1.1) — the plain method is not an acceptable substitute where SHA-256 is available.
- **Verify support before proceeding.** Because neither OAuth 2.1 nor PKCE defines a mechanism to discover PKCE support, clients MUST confirm it from authorization-server metadata and **refuse to proceed** if they cannot:

| Metadata source | Rule |
| --- | --- |
| [[rfc-8414-authorization-server-metadata|OAuth 2.0 AS Metadata]] | If `code_challenge_methods_supported` is **absent**, the AS does not support PKCE; the client MUST refuse to proceed. |
| [[openid-connect-discovery|OIDC Discovery 1.0]] | The field is not defined by OIDC metadata but is commonly present; the client MUST verify its presence and MUST refuse to proceed if absent. ASes offering OIDC Discovery MUST include it for MCP compatibility. |

This is a **downgrade-attack defense**: refusing to continue when PKCE support is unverifiable prevents an attacker or a misconfigured AS from silently stripping the protection that makes code interception harmless.

## Relation to pre-AI IAM

This is verbatim **RFC 7636**, elevated to mandatory by [[oauth-2-1]] §4.1.1. Any practitioner who has shipped a mobile or SPA OAuth client in the last several years already uses it; the mental model ("hash now, reveal at redemption") is unchanged. See [[rfc-7636-pkce]].

## Why pre-AI IAM is insufficient

PKCE itself needs no agentic adaptation — it is sufficient and is simply *required* rather than recommended here. The agentic relevance is contextual: MCP clients are overwhelmingly public clients operating in an environment with untrusted content and dynamically discovered servers, so the threats PKCE addresses (code interception on the redirect leg) are more likely, and the spec removes the option to skip it. This is the same "promote optional hardening to mandatory" pattern seen across [[mcp-authorization]].
