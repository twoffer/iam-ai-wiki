---
title: Delegated Authorization
category: concept
status: stable
confidence: high
aliases: ["delegation", "on-behalf-of authorization", "resource owner delegation"]
enterprise_analogs: ["OAuth 2.1 authorization code grant", "RFC 6749 §1.1 resource owner / client", "RFC 8693 token exchange", "OAuth 2.0 on-behalf-of"]
last_updated: 2026-07-14
sources: ["mcp-authorization-overview", "owasp-llm-top-10-2025"]
related: ["mcp-authorization", "agentic-identity", "human-in-the-loop-authorization", "tool-use-authorization", "token-audience-binding", "step-up-authorization", "excessive-agency", "machine-identity"]
tags: ["oauth", "delegation", "authorization", "core-concept"]
---

# Delegated Authorization

**Delegated authorization** is the pattern in which a software agent acts on a protected resource *on behalf of* a resource owner (typically a human user), using credentials the owner consented to grant — rather than the agent's own standalone authority. It is the foundational shape of OAuth and the foundational shape of [[mcp-authorization]].

In MCP terms ([[mcp-authorization-overview]]): the **MCP client** "make[s] protected resource requests on behalf of a resource owner," and the **authorization server** "interact[s] with the user (if necessary) and issu[es] access tokens." The user delegates a bounded slice of their authority (expressed as scopes) to the client, which presents the resulting token to the [[mcp-authorization|MCP server]].

## Key properties

- **Bounded.** The delegation is limited to the granted scopes (see [[scope-selection-strategy]]), following least privilege; it can be widened later via [[step-up-authorization]].
- **Consented.** A user authorization/consent step (the browser leg of the flow) records the owner's approval. See [[human-in-the-loop-authorization]].
- **Audience-restricted.** The delegated token is bound to a specific resource ([[token-audience-binding]]) so it cannot be reused elsewhere.
- **Revocable and expiring.** Tokens expire; refresh tokens (when issued) let the client maintain delegated access without re-prompting.

## Delegation vs. the standing service identity

The [[owasp-llm-top-10|OWASP LLM Top 10]] makes user delegation a named mitigation for agentic systems: [[excessive-agency|LLM06]] requires that extensions "execute in the user's context" — actions on downstream systems run under the specific requesting user's authority with minimum privileges, "for example, an LLM extension that reads a user's code repo should require the user to authenticate via OAuth and with the minimum scope required." The corresponding anti-pattern is *excessive permissions*: a per-user operation performed under "a generic high-privileged identity" with access to every user's data — a broad [[machine-identity]] silently substituting for a narrow user delegation ([[owasp-llm-top-10-2025]], LLM06 *Common Examples*, *Prevention*).

## Relation to pre-AI IAM

This is the original OAuth use case. The **authorization code grant** ([[oauth-2-1]], historically RFC 6749 §4.1) exists precisely to let a third-party app act for a user without handling the user's password. The mental model carries over unchanged: resource owner, client, authorization server, resource server (RFC 6749 §1.1). Enterprise patterns for *re-delegation* down a call chain — [[token-audience-binding|RFC 8693 token exchange]], the "on-behalf-of" flow — are the pre-AI analog of propagating a user's delegated authority through multiple services.

## Why pre-AI IAM is insufficient

Traditional delegation assumes a **single, statically known delegate** integrated by a developer: app X is authorized to call API Y, and the set {X, Y} is fixed at integration time. Agentic delegation differs:

- **The delegate is dynamic and compositional.** An agent may delegate onward to tools, sub-agents, and servers chosen at runtime (see [[agentic-identity]] and [[tool-use-authorization]]). Each hop needs its own bounded, audience-bound token rather than a reused bearer token — otherwise delegation silently becomes impersonation.
- **The delegate can be steered by untrusted input.** Because an LLM agent's behavior is influenced by content it processes, a delegated token in its possession is a higher-value target for [[confused-deputy]] abuse. This is why MCP promotes audience binding and the [[token-passthrough]] prohibition from best practice to requirement.
- **Scope cannot be fully predetermined.** A general-purpose agent does not know in advance which permissions a task will need, so delegation must be negotiated incrementally at runtime ([[step-up-authorization]]) rather than fixed at registration.
