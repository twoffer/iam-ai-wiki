---
title: Prompt Injection as Authorization Bypass
category: concept
status: stub
confidence: medium
aliases: ["prompt injection", "indirect prompt injection", "injection-driven authorization bypass"]
enterprise_analogs: ["SQL injection (in-band control/data confusion)", "CSRF (attacker-directed use of a victim's authority)", "confused deputy"]
last_updated: 2026-07-08
sources: ["mcp-security-best-practices"]
related: ["confused-deputy", "agentic-identity", "session-hijacking", "tool-use-authorization", "human-in-the-loop-authorization", "security-considerations"]
tags: ["prompt-injection", "security", "agentic", "auth-bypass", "stub"]
---

# Prompt Injection as Authorization Bypass

**Prompt injection** is untrusted content steering an agent's behavior — and in an IAM frame, its significance is that the agent then exercises its *legitimate, delegated authority* on the attacker's behalf. No credential is stolen and no protocol control fails; the [[confused-deputy|deputy]] is redirected through its input channel, making injection an authorization-bypass vector that classic token- and consent-level controls cannot see.

## Sourced touchpoints so far

- The MCP Authorization spec's normative [[security-considerations|Security Considerations]] do **not** treat prompt injection; the wiki flags it there as the unaddressed agentic wrinkle.
- The [[mcp-security-best-practices|Security Best Practices]] guide provides the first concrete, infrastructure-level delivery vector in the corpus: **[[session-hijacking|session hijack prompt injection]]**, where an attacker writes into a session-keyed event queue and the victim's client acts on the injected payload — including silently acquiring server tools via a forged `notifications/tools/list_changed` (see [[tool-use-authorization]]).
- The pre-AI analogs are in-band control/data confusion (SQL injection) and attacker-directed use of standing authority (CSRF); the agentic difference is that the "interpreter" is a non-deterministic model and the "query" is all content the agent reads.

> **Status: stub.** Seeded from the confused-deputy analysis and the Security Best Practices guide's session-hijack vector. Awaiting dedicated sources (e.g., OWASP LLM Top 10 LLM01, indirect-prompt-injection research, tool-poisoning write-ups) for a full treatment of injection-resistant authorization design ([[human-in-the-loop-authorization]], capability confinement, output mediation).
