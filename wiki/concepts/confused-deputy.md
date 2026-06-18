---
title: Confused Deputy
category: concept
status: evolving
confidence: high
aliases: [confused deputy problem, confused deputy attack, deputy confusion]
enterprise_analogs: [OAuth 2.0 mix-up attack, CSRF, RFC 9700 OAuth 2.0 Security BCP, ambient authority]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [token-audience-binding, token-passthrough, authorization-server-mix-up, mcp-authorization, security-considerations, agentic-identity, delegated-authorization]
tags: [security, confused-deputy, threat-model, core-concept]
---

# Confused Deputy

A **confused deputy** is a privileged component tricked into misusing its authority on behalf of a less-privileged or malicious party. The deputy is not compromised in the sense of running attacker code; it is *confused* into applying its legitimate credentials to an attacker-chosen request. The term predates AI (Norm Hardy, 1988), but agentic systems make it a central threat: an AI agent is, almost by definition, a deputy that wields delegated authority across many tools and servers.

## In agentic / MCP contexts

An MCP client holds tokens delegated by a user and connects to multiple servers. Two confused-deputy shapes matter:

- **Cross-resource token misuse.** If a token were not [[token-audience-binding|audience-bound]], a malicious server (or an instruction injected into content the agent processes) could induce the client to present a token — minted for server A — to server B, exercising the user's authority where it was never granted. MCP's mandatory audience binding and [[token-passthrough]] prohibition are the structural fix.
- **Authorization-server mix-up.** A client tricked into talking to the wrong AS, or redeeming a code at the wrong endpoint, can leak a code/token to an attacker. MCP's mandatory [[rfc-9207-authorization-server-issuer-identification|`iss` validation]] and discovery rules defend this; see [[authorization-server-mix-up]].

The [[mcp-authorization-overview]] explicitly lists "mix-up and confused deputy attacks" among the threats its [[security-considerations]] chapter addresses normatively.

## Relation to pre-AI IAM

This is a well-known OAuth threat class. The **OAuth 2.0 mix-up attack**, **CSRF** on the redirect, and **ambient-authority** abuse are all confused-deputy instances, catalogued in the OAuth 2.0 Security Best Current Practice (RFC 9700). The standard mitigations — exact redirect-URI matching, PKCE, `state`, audience restriction, issuer identification — are the same primitives MCP mandates. A practitioner's existing confused-deputy mental model applies directly.

## Why pre-AI IAM is insufficient

The classic mitigations assume a **fixed, human-integrated topology** and a deputy whose behavior is deterministic code. Agentic systems break both:

- The deputy's control flow is **influenced by untrusted natural-language input** (prompt injection), so an attacker can steer *which* request the deputy makes, not just intercept it. Authority confusion can be induced through data, not only through protocol manipulation.
- The deputy brokers **many dynamically discovered servers**, so the population of potential victims/attackers is open and unbounded.

This is why MCP promotes optional OAuth hardening (audience binding, issuer validation, no token passthrough) to **mandatory**, and why prompt-injection-as-authorization-bypass is a first-class agentic concern beyond anything the pre-AI threat model required.

> **Status:** evolving. Expanded coverage expected when [[security-considerations]] (raw doc 4) is ingested.
