---
title: Excessive Agency
category: concept
status: stable
confidence: high
aliases: ["LLM06:2025", "excessive functionality", "excessive permissions", "excessive autonomy", "over-permissioned agent"]
enterprise_analogs: ["over-privileged service accounts", "OAuth over-scoping (RFC 6749 §3.3)", "least privilege / complete mediation (Saltzer–Schroeder 1975)", "RBAC role explosion and privilege creep", "change-approval workflows"]
last_updated: 2026-07-14
sources: ["owasp-llm-top-10-2025"]
related: ["tool-use-authorization", "delegated-authorization", "human-in-the-loop-authorization", "confused-deputy", "prompt-injection", "scope-selection-strategy", "machine-identity", "agentic-identity", "system-prompt-leakage"]
tags: ["agentic", "least-privilege", "authorization", "owasp", "core-concept"]
---

# Excessive Agency

**Excessive agency** is the vulnerability class in which an LLM-based system is granted more capability, permission, or autonomy than its intended operation requires, so that unexpected, ambiguous, or manipulated model outputs translate into damaging real-world actions. The term is OWASP's (LLM06 of the [[owasp-llm-top-10|OWASP Top 10 for LLM Applications]], expanded in the 2025 revision specifically for agentic architectures); the defining property is cause-independence — the vulnerability "enables damaging actions to be performed … **regardless of what is causing the LLM to malfunction**," whether hallucination from a benign prompt, direct or indirect [[prompt-injection]], a compromised extension, or a malicious peer agent in a multi-agent system ([[owasp-llm-top-10-2025]], LLM06 *Description*).

Agency here means the ability to call functions or interface with other systems via *extensions* (vendor vocabulary varies: Anthropic and MCP say "tools," others say "skills" or "plugins"), possibly with the LLM itself deciding which extension to invoke based on prior outputs. Excessive agency is what turns an injectable model into an exploitable [[confused-deputy]]: injection supplies the confusion, agency supplies the authority.

## The three root causes

OWASP decomposes the vulnerability into three independently fixable properties ([[owasp-llm-top-10-2025]], LLM06):

1. **Excessive functionality** — the agent can invoke operations its task never needs: a third-party extension chosen for document *reading* that also exposes modify-and-delete, a plugin left enabled after the development phase that trialled it, or an open-ended extension (run a shell command, fetch a URL) where a narrow one would do.
2. **Excessive permissions** — the identity an extension presents downstream carries more privilege than the operation requires: a read-oriented extension whose database identity also holds UPDATE/INSERT/DELETE, or a per-user operation executed under "a generic high-privileged identity" with access to every user's data — the [[machine-identity]]-substituting-for-[[delegated-authorization|user delegation]] failure.
3. **Excessive autonomy** — high-impact actions execute without independent verification or approval: deletion without confirmation, sending without review. The missing control is [[human-in-the-loop-authorization]].

## The mitigation stack, in IAM terms

Every LLM06 mitigation is a classical authorization control relocated around the model ([[owasp-llm-top-10-2025]], LLM06 *Prevention*):

- **Minimize extensions and extension functionality** — offer the agent only the tools the task needs, and build tools that implement only the needed operations ([[tool-use-authorization]]). Avoid open-ended extensions in favor of granular ones (a file-writing tool, not a shell tool).
- **Minimize extension permissions** — the downstream identity behind each tool is scoped to the minimum (database grants, API scopes), enforced *by the downstream system's own permission model* (see [[scope-selection-strategy]] for the OAuth-scope expression of the same rule).
- **Execute extensions in the user's context** — actions on downstream systems run under the specific requesting user's authority with minimum privileges, e.g. a user-authenticated OAuth session with the minimum scope required, not a standing service identity ([[delegated-authorization]]).
- **Require user approval** — human-in-the-loop confirmation before high-impact actions, implementable in the downstream system or in the extension that performs the operation ([[human-in-the-loop-authorization]]).
- **Complete mediation** — "implement authorization in downstream systems rather than relying on an LLM to decide if an action is allowed"; every request an extension makes downstream is validated against security policy. This is the same rule [[system-prompt-leakage|LLM07]] states from the other direction: authorization checks must not be delegated to the model.
- **Sanitize LLM inputs and outputs** — standard ASVS discipline, acknowledging the model boundary as an untrusted interface.

Logging/monitoring of extension activity and rate limiting are explicitly classed as damage *limiters* that "will not prevent" excessive agency — detection and blast-radius controls, not authorization.

OWASP's worked scenario is the canonical agentic confused deputy: an email-assistant extension needs read access to a mailbox, but the chosen plugin also sends mail (excessive functionality), authenticates with broad standing credentials (excessive permissions), and acts on what it reads without review (excessive autonomy) — so an indirect injection in an incoming email can command it to forward the inbox's sensitive content to the attacker. Each root cause is fixed independently: a read-only extension, a read-only OAuth scope, and a human review before every send ([[owasp-llm-top-10-2025]], LLM06 *Example Attack Scenarios*).

## Relation to pre-AI IAM

This is the least-privilege audit every IAM practitioner has run, with the LLM application in the role of the over-privileged service account. Excessive functionality is unused-entitlement sprawl; excessive permissions is the account holding `db_owner` when it needs `SELECT`, or a daemon running as root; excessive autonomy is a pipeline that deploys without a change-approval gate. The mitigations are Saltzer–Schroeder verbatim — least privilege and complete mediation — plus OAuth scope minimization (RFC 6749 §3.3) for the delegated case. The mental model "never let the app decide its own authorization; the resource server decides" carries over unchanged.

## Why pre-AI IAM is insufficient

Pre-AI over-permissioning was a *latent* risk: the deputy was deterministic code, so unused privilege was only dangerous if the code was compromised or buggy. An LLM agent's control flow is steered by every piece of content it processes, so unused privilege is **continuously reachable by anyone who can get text in front of the model** — over-permissioning converts directly into attacker capability, no code compromise required. That is why OWASP frames the vulnerability as cause-independent and why the controls are structural (capability, permission, autonomy bounds) rather than input-side: [[prompt-injection]] cannot be reliably prevented, so the authority available to a manipulated model must be small enough that manipulation doesn't matter. Pre-AI IAM also never had to authorize a *dynamically chosen* action set — an agent selects tools and arguments at runtime, which pushes enforcement to per-call mediation in downstream systems rather than integration-time entitlement review.
