---
title: Machine Identity (for AI Agents)
category: concept
status: stub
confidence: medium
aliases: [machine identity, workload identity, non-human identity, NHI]
enterprise_analogs: [RFC 6749 §4.4 client_credentials, mutual TLS / X.509, SPIFFE/SPIRE SVID, API keys, service accounts]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [agentic-identity, delegated-authorization, mcp-authorization, tool-use-authorization]
tags: [machine-identity, workload, agentic, stub]
---

# Machine Identity (for AI Agents)

**Machine identity** is the identity an agent or workload holds *as itself*, distinct from any user authority it carries on behalf of others ([[delegated-authorization]]). It answers "which piece of software is this?" rather than "whose permission is this acting under?" — the two halves of [[agentic-identity]].

## Touchpoint in the MCP Authorization spec

The [[mcp-authorization-overview]] mostly concerns *delegated user* authority, but it touches machine identity at two points:

- **STDIO transport.** Implementations using a STDIO transport "SHOULD NOT" follow the OAuth flow and instead "retrieve credentials from the environment" (*Protocol Requirements*) — i.e., a workload/service credential supplied out of band, not a user-delegated token.
- **`client_credentials` clients.** The [[step-up-authorization]] section distinguishes clients "acting on their own behalf (`client_credentials` clients)" from clients acting for a user — the former use a pure machine identity (RFC 6749 §4.4).

## Relation to pre-AI IAM

Standard ground: a service's own identity is `client_credentials`, mutual TLS / X.509 client certs, API keys, cloud service accounts, or a SPIFFE SVID. Nothing here is AI-specific yet.

## Why pre-AI IAM is insufficient

The open questions — attesting that a credential belongs to a *specific model/agent build*, binding a short-lived workload identity to a delegated user grant, and preventing an agent's broad machine identity from substituting for a narrow user delegation — are agentic extensions not yet settled by this spec. See [[agentic-identity]].

> **Status:** stub. Seeded from the MCP overview's STDIO and `client_credentials` mentions; to be expanded by future workload-identity / SPIFFE / attestation sources.
