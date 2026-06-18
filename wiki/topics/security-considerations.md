---
title: MCP Authorization — Security Considerations
category: topic
status: stub
confidence: high
aliases: [MCP security considerations, MCP authorization security]
enterprise_analogs: [RFC 9700 OAuth 2.0 Security BCP, RFC 6819 OAuth threat model, RFC 9207 issuer identification, RFC 8707 audience restriction]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, token-audience-binding, token-passthrough, confused-deputy, authorization-server-mix-up, proof-key-for-code-exchange, oauth-client-id-metadata-documents]
tags: [security, threat-model, mcp, stub]
---

# MCP Authorization — Security Considerations

The fourth document of the MCP Authorization specification defines the **normative security requirements** that implementations MUST follow. The [[mcp-authorization-overview|overview]] defers to it and enumerates the areas it covers.

> **Status: stub.** The dedicated document (`raw/MCPAuthorization_SecurityConsiderations.md`) is present but **not yet ingested**. This page records the threat areas the overview names so links resolve; it will be replaced with full normative detail on ingest.

## Threat areas named in the overview

Per *Security Considerations* in [[mcp-authorization-overview]], implementations MUST follow normative requirements covering:

- **Token audience binding and validation** — see [[token-audience-binding]], [[canonical-server-uri]].
- **Token theft** — protecting tokens in transit and storage.
- **Communication security** — TLS and related transport protections.
- **Authorization code protection** — see [[proof-key-for-code-exchange|PKCE]].
- **Mix-up and confused deputy attacks** — see [[authorization-server-mix-up]], [[confused-deputy]].
- **Open redirection** — strict `redirect_uri` handling.
- **Client ID Metadata Document security** — see [[oauth-client-id-metadata-documents]].

## Relation to pre-AI IAM

These map almost one-to-one onto the **OAuth 2.0 Security Best Current Practice (RFC 9700)** and the original threat model (RFC 6819): audience restriction, PKCE, exact redirect-URI matching, issuer identification, and transport security. A practitioner's OAuth threat model transfers; the MCP document tightens which mitigations are mandatory.

## Why pre-AI IAM is insufficient

The agentic deployment model — open client/server populations discovered at runtime, plus agents steerable by untrusted input — raises the likelihood and impact of these classic threats, which is why MCP elevates several optional OAuth hardening measures to requirements. The specific new wrinkle is **prompt injection as an authorization-bypass vector**, where untrusted content manipulates the deputy itself; the degree to which the dedicated document addresses this will be captured on ingest.
