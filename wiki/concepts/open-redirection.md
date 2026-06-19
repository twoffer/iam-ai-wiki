---
title: Open Redirection
category: concept
status: stable
confidence: high
aliases: [open redirect, open redirection, redirect_uri attack, redirection attack]
enterprise_analogs: [OAuth 2.1 §7.12.2 open redirection, exact redirect-URI matching, RFC 9700 OAuth 2.0 Security BCP, OAuth state parameter / CSRF]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations]
related: [mcp-authorization, security-considerations, authorization-server-mix-up, confused-deputy, oauth-client-id-metadata-documents, oauth-2-1]
tags: [security, oauth, redirect, threat-model, core-concept]
---

# Open Redirection

**Open redirection** is the threat that an attacker crafts a malicious `redirect_uri` (or abuses a lax one) to send the user agent to an attacker-controlled site — typically to phish credentials or to capture an authorization code on the redirect leg. In OAuth the redirect endpoint is where the authorization code lands, so loose redirect handling is both a phishing vector and a code-interception vector ([[mcp-authorization-security-considerations]], *Open Redirection*).

## How MCP mitigates it

- **Registered redirect URIs (MUST).** MCP clients MUST have their redirect URIs registered with the authorization server.
- **Exact matching (MUST).** ASes MUST validate **exact** redirect URIs against the pre-registered values — no prefix/substring matching, which is the classic open-redirect hole.
- **`state` parameter (SHOULD).** Clients SHOULD use and verify `state` in the authorization-code flow and discard any result that is missing `state` or whose `state` does not match the value they sent. (This is also the CSRF defense for the redirect.)
- **Untrusted-URI precautions (MUST / SHOULD).** ASes MUST take precautions against redirecting user agents to untrusted URIs ([[oauth-2-1|OAuth 2.1]] §7.12.2), and SHOULD only auto-redirect to a redirection URI they trust — otherwise informing the user and relying on the user to decide.

Exact redirect-URI matching also underpins the [[oauth-client-id-metadata-documents|CIMD]] flow, where the AS validates the request's `redirect_uri` against the `redirect_uris` in the fetched metadata document; the `localhost`-impersonation caveat ([[security-considerations]]) is the place where exact matching is necessary but not sufficient.

## Relation to pre-AI IAM

This is the textbook OAuth **open-redirect / redirect-URI** threat and its standard mitigations: pre-registration, exact matching, and `state` — all catalogued in the OAuth 2.0 Security BCP (RFC 9700) and required by [[oauth-2-1|OAuth 2.1]] §7.12.2. A practitioner who has hardened an OAuth redirect endpoint already knows every control here; nothing about the mechanism changes.

## Why pre-AI IAM is insufficient

The controls are unchanged; the **population they must hold against is open**. In classic OAuth a human pre-registers a small set of exact redirect URIs for a known app. In MCP, clients are discovered at runtime and may legitimately use `localhost` redirect URIs (native apps, CLIs), and with [[oauth-client-id-metadata-documents|CIMD]] the client's identity and its declared redirect URIs are self-asserted via a fetched document. Exact matching still applies, but it cannot distinguish a legitimate `localhost` client from an attacker who claims the same metadata URL and binds a `localhost` port (see the `localhost` impersonation case in [[security-considerations]]). MCP therefore keeps the classic redirect controls mandatory **and** adds AS-side display/warning requirements so the human in the loop can catch what exact matching cannot.
