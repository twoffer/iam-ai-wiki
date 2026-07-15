---
title: Prompt Injection as Authorization Bypass
category: concept
status: evolving
confidence: high
aliases: ["prompt injection", "direct prompt injection", "indirect prompt injection", "injection-driven authorization bypass", "jailbreaking", "LLM01:2025"]
enterprise_analogs: ["SQL injection (in-band control/data confusion)", "CSRF (attacker-directed use of a victim's authority)", "confused deputy", "parameterized queries (the missing analog)"]
last_updated: 2026-07-14
sources: ["mcp-security-best-practices", "owasp-llm-top-10-2025"]
related: ["confused-deputy", "excessive-agency", "agentic-identity", "session-hijacking", "tool-use-authorization", "human-in-the-loop-authorization", "system-prompt-leakage", "vector-store-access-control", "security-considerations"]
tags: ["prompt-injection", "security", "agentic", "auth-bypass", "owasp"]
---

# Prompt Injection as Authorization Bypass

**Prompt injection** is untrusted content steering an agent's behavior — and in an IAM frame, its significance is that the agent then exercises its *legitimate, delegated authority* on the attacker's behalf. No credential is stolen and no protocol control fails; the [[confused-deputy|deputy]] is redirected through its input channel, making injection an authorization-bypass vector that classic token- and consent-level controls cannot see.

## The OWASP definition (LLM01)

The [[owasp-llm-top-10|OWASP LLM Top 10]] ranks prompt injection first: inputs that "alter the LLM's behavior or output in unintended ways," which need not be human-visible or readable — only parsed by the model ([[owasp-llm-top-10-2025]], LLM01 *Description*). Its taxonomy:

- **Direct injection** — the user's own prompt alters behavior, whether deliberately (a crafted exploit) or unintentionally (input that happens to trigger unexpected behavior).
- **Indirect injection** — the model ingests external content (websites, files, email, retrieved documents) carrying instructions. This is the vector that matters most for agents, since an agent's job is to read things.
- **Jailbreaking** is classified as a form of prompt injection whose goal is making the model disregard its safety protocols entirely.

Among OWASP's enumerated impacts, three are authorization outcomes: *providing unauthorized access to functions available to the LLM*, *executing arbitrary commands in connected systems*, and *disclosure of sensitive information* (including AI-system infrastructure and [[system-prompt-leakage|system prompts]]). Severity is "largely dependent on … the agency with which the model is architected" — injection supplies the confusion, [[excessive-agency]] supplies the authority.

## Injection-resistant authorization design

OWASP concedes the decisive point: "given the stochastic influence at the heart of the way models work, it is unclear if there are fool-proof methods of prevention for prompt injection" ([[owasp-llm-top-10-2025]], LLM01 *Prevention*). Its mitigation list therefore splits into model-side conditioning that reduces likelihood (constrain model behavior via system prompt, define and validate output formats, semantic input/output filtering) and **authorization-grade controls that assume injection succeeds**:

- **Privilege control and least privilege.** The application holds its own API tokens for extensible functionality and handles those functions "in code rather than providing them to the model"; the model's access is restricted to the minimum its operation requires. Credentials stay out of the model's reach entirely — see [[tool-use-authorization]] and [[system-prompt-leakage]].
- **Human approval for high-risk actions.** [[human-in-the-loop-authorization|Human-in-the-loop]] controls for privileged operations, so a steered agent cannot complete a damaging action alone.
- **Segregate and identify external content.** Untrusted content is separated and marked to limit its influence on user prompts — the closest available gesture toward control/data separation.
- **Treat the model as an untrusted user.** Adversarial testing and penetration tests model the LLM itself as an untrusted party, probing whether trust boundaries and access controls hold when the model misbehaves.

The same design conclusion appears in LLM07's rule that privilege separation and authorization bounds checks must never be delegated to the model ([[system-prompt-leakage]]) and LLM06's complete-mediation requirement ([[excessive-agency]]): because the input channel cannot be sealed, enforcement must live outside the model.

## Sourced delivery vectors

- **MCP session-hijack injection.** The [[mcp-security-best-practices|MCP Security Best Practices]] guide provides an infrastructure-level vector: an attacker writes into a session-keyed event queue and the victim's client acts on the injected payload — including silently acquiring server tools via a forged `notifications/tools/list_changed` (see [[session-hijacking]], [[tool-use-authorization]]).
- **The retrieval corpus.** RAG documents are an indirect-injection channel: OWASP's scenarios include an attacker-modified repository document that alters outputs when retrieved, and a résumé with hidden white-on-white instructions steering a RAG screening system ([[vector-store-access-control]]).
- **Application-level exploits.** CVE-2024-5184, prompt injection in an LLM email assistant giving access to sensitive information and mail manipulation; a support chatbot directly injected into ignoring guidelines, querying private data stores, and sending email — "unauthorized access and privilege escalation" ([[owasp-llm-top-10-2025]], LLM01 *Example Attack Scenarios*).
- **Evasion encodings.** Multimodal injection (instructions hidden in images accompanying benign text, with cross-modal attacks flagged as hard to detect), payload splitting across multiple inputs, adversarial suffixes, and multilingual/Base64/emoji obfuscation defeat input filters — reinforcing that filtering is a likelihood reducer, not a boundary.

## Relation to pre-AI IAM

The pre-AI analogs are in-band control/data confusion (SQL injection) and attacker-directed use of standing authority (CSRF); both are [[confused-deputy]] instances, and the practitioner instincts — validate input, but ultimately re-authorize at the resource — carry over. OWASP's "treat the model as an untrusted user" is the zero-trust posture practitioners already apply to any client-supplied request.

## Why pre-AI IAM is insufficient

SQL injection was *solved* by parameterized queries: a deterministic mechanism that separates control channel from data channel. Natural language has no such mechanism — the model's instructions and its data arrive in the same token stream, the interpreter is probabilistic, and OWASP explicitly declines to promise preventability. Authorization design must therefore invert: assume the agent's instructions can be attacker-influenced at any time, and bound what a subverted agent can do via least privilege, per-call downstream mediation, and human approval — controls that make injection survivable rather than impossible. That inversion, and the fact that the deputy's entire input surface (every document, email, and tool result it reads) is the attack surface, have no pre-AI counterpart.
