---
title: Agentic Identity
category: concept
status: evolving
confidence: medium
aliases: ["agent identity", "AI agent identity", "agentic IAM"]
enterprise_analogs: ["OAuth 2.1 client / resource owner separation", "RFC 6749 delegated authorization", "RFC 8693 token exchange", "workload identity (SPIFFE)"]
last_updated: 2026-07-08
sources: ["mcp-authorization-overview"]
related: ["delegated-authorization", "machine-identity", "mcp-authorization", "tool-use-authorization", "human-in-the-loop-authorization", "confused-deputy", "token-audience-binding", "prompt-injection"]
tags: ["agentic", "identity", "core-concept", "domain-anchor"]
---

# Agentic Identity

**Agentic identity** is the umbrella concept for *who an AI agent is* and *whose authority it carries* when it takes actions against protected systems. It spans the agent's own identity (a workload/[[machine-identity|machine identity]]), the identity it acts on behalf of (a delegating user — see [[delegated-authorization]]), and how those are bound, propagated, and constrained across tool calls and agent-to-agent hops.

This page is the **domain anchor** for the wiki. Most other pages are facets of it: how an agent obtains delegated authority ([[mcp-authorization]]), how it proves which authority a given credential carries ([[token-audience-binding]]), what it is allowed to do with a tool ([[tool-use-authorization]]), and when a human must approve ([[human-in-the-loop-authorization]]).

## Two identities in play

An agent action typically involves **two distinct identities** that pre-AI IAM also recognizes but rarely had to compose so dynamically:

1. **The principal being acted for** — the human resource owner who delegated authority. Carried as a user-consented OAuth token in MCP ([[delegated-authorization]]).
2. **The agent/workload itself** — the running client software, which may also have its own identity for non-delegated (`client_credentials`) actions or for attestation. See [[machine-identity]].

Conflating these — letting the agent's broad standing authority stand in for a narrowly delegated user grant, or vice versa — is the root of many agentic auth failures, including [[confused-deputy]] patterns.

## How the MCP Authorization spec touches this

The [[mcp-authorization-overview]] does not define "agentic identity" as a term, but it operationalizes it: the MCP client is an OAuth 2.1 client acting for a resource owner, every token is bound to a specific resource audience, and the spec explicitly distinguishes clients "acting on behalf of a user" from clients "acting on their own behalf (`client_credentials` clients)" in the [[step-up-authorization]] discussion. That user-vs-self distinction is the agentic-identity question made concrete.

## Relation to pre-AI IAM

The constituent pieces are all standard: delegated user authority is OAuth ([[oauth-2-1]]); a service's own identity is `client_credentials` (RFC 6749 §4.4) or a workload identity such as SPIFFE; propagating a user's identity across services is [[token-audience-binding|token exchange (RFC 8693)]]. A practitioner already has every primitive needed.

## Why pre-AI IAM is insufficient

What is new is the **composition and dynamism**: an agent assembles chains of delegated and self-authority at runtime, across services chosen on the fly, while being influenceable by untrusted data. Pre-AI IAM assembled these chains *statically*, at integration time, under human control. The open problems — propagating identity through agent chains, attesting AI-generated tokens, preventing [[prompt-injection|prompt-injection-driven]] authority confusion — arise because the binding between "who is acting" and "whose authority is used" must now be maintained automatically, per request, in an adversarial input environment. The MCP profile is one early, scoped answer (see [[mcp-authorization|Why pre-AI IAM is insufficient]] there); broader agentic-identity standards are still emerging.

> **Status:** evolving. Seeded from the MCP Authorization overview; expected to deepen as agent-to-agent authentication, attestation, and workload-identity sources are ingested.
