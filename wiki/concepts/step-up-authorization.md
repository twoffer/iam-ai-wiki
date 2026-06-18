---
title: Step-Up Authorization
category: concept
status: stable
confidence: high
aliases: [step-up auth, incremental authorization, scope step-up, insufficient_scope handling]
enterprise_analogs: [RFC 6750 §3.1 insufficient_scope, OAuth incremental authorization, RFC 9470 step-up authentication challenge, OIDC claims request]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [scope-selection-strategy, mcp-authorization, tool-use-authorization, delegated-authorization, human-in-the-loop-authorization]
tags: [oauth, scopes, step-up, authorization, core-concept]
---

# Step-Up Authorization

**Step-up authorization** is the runtime escalation of a client's permissions: a client that already holds a token but lacks sufficient scope for an operation obtains a *new* token carrying additional scopes, then retries. In MCP it is the mechanism by which a general-purpose agent acquires only the permissions a task actually needs, incrementally, instead of requesting everything up front ([[mcp-authorization-overview]], *Step-Up Authorization Flow*).

## How it works in MCP

1. The server rejects an under-scoped request with **`403 Forbidden`** + `WWW-Authenticate: Bearer error="insufficient_scope"`, including a `scope` challenge listing the minimum scopes for the operation and `resource_metadata` (*Runtime Insufficient Scope Errors*).
2. The client **parses** the challenge, then **computes the union** of its previously requested scopes and the newly challenged scopes — so prior permissions are not lost (see [[scope-selection-strategy]]).
3. The client **re-authorizes** with the unioned scope set and **retries** the original request, "no more than a few times," with retry limits, treating repeated failure as permanent.

**Scope accumulation is a client-side responsibility.** Servers stay stateless about client scope sets: they emit only the scopes needed for the current operation and need not echo previously granted ones. Clients acting for a user SHOULD attempt step-up; `client_credentials` clients MAY step up or abort immediately.

Servers SHOULD challenge with *all* scopes a single operation needs at once (not one at a time), and MAY bundle related/anticipated scopes to cut the number of round-trips — trading breadth against authorization friction. Hierarchical scopes (e.g., `admin` implying `read`) need not be deduplicated by the client; the AS normalizes redundancy at issuance.

## Relation to pre-AI IAM

The wire mechanics are standard OAuth: the `insufficient_scope` error and `WWW-Authenticate` challenge are [[rfc-6750-bearer-token-usage|RFC 6750]] §3.1; the pattern is **incremental authorization**, long used by APIs (e.g., Google) to request scopes lazily, and is conceptually adjacent to **step-up authentication** (RFC 9470) where an operation demands a stronger/fresher credential. A practitioner's "challenge → re-consent for more scope → retry" model transfers directly.

## Why pre-AI IAM is insufficient

Traditional apps are **domain-aware**: a developer knows the exact scope set at build time and requests it once. A general-purpose MCP client has "no domain-specific knowledge to make informed decisions about individual scope selection" ([[mcp-authorization-overview]], *Scope Selection Strategy*). It therefore cannot pre-declare scopes correctly and must negotiate them at runtime, per operation, often mid-task as the agent decides which [[tool-use-authorization|tool]] to invoke. Step-up turns scope acquisition from a one-time design decision into a continuous runtime protocol — necessary precisely because the agent, not a human integrator, is choosing the actions.
