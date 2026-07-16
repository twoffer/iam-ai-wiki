---
title: System Prompt Leakage
category: concept
status: stable
confidence: high
aliases: ["LLM07:2025", "system prompt disclosure", "prompt leakage", "meta prompt extraction"]
enterprise_analogs: ["hard-coded credentials (CWE-798)", "secrets in client-side code (JS bundles, mobile binaries)", "client-side authorization checks", "security through obscurity"]
last_updated: 2026-07-14
sources: ["owasp-llm-top-10-2025"]
related: ["prompt-injection", "excessive-agency", "token-theft", "tool-use-authorization", "human-in-the-loop-authorization", "agentic-identity"]
tags: ["agentic", "security", "secrets", "owasp", "threat-model"]
---

# System Prompt Leakage

**System prompt leakage** is the risk that the instructions used to steer an LLM's behavior are discovered by users or attackers, exposing whatever sensitive information or security-relevant logic was embedded in them. The entry is new in the 2025 [[owasp-llm-top-10|OWASP LLM Top 10]] (LLM07), added because "many applications assumed prompts were securely isolated" and real-world exploits showed otherwise ([[owasp-llm-top-10-2025]], *Letter from the Project Leads*).

The IAM significance is the entry's central negative claim: "the system prompt **should not be considered a secret, nor should it be used as a security control**." Disclosure of the prompt's wording is not itself the vulnerability — attackers interacting with any deployed model can reconstruct most of its guardrails from observed behavior anyway. The real risks are the *underlying elements* the leak reveals: credentials that should never have been in the prompt, and authorization logic that should never have been delegated to the model ([[owasp-llm-top-10-2025]], LLM07 *Description*).

## What leaks, and why it matters

OWASP's four risk examples ([[owasp-llm-top-10-2025]], LLM07 *Common Examples*):

1. **Sensitive functionality and credentials** — API keys, database connection strings, user tokens placed in the prompt for tool access. A leaked prompt hands them to the attacker directly (scenario #1 is exactly this; see [[token-theft]] — the system prompt is a token-storage location that fails every OAuth 2.1 §7.1 storage requirement).
2. **Internal rules** — business limits ("transaction limit is $5,000/day") that let an attacker calibrate transactions to bypass controls the application enforces only via the prompt.
3. **Filtering criteria** — the model's instructed refusal rules, which the attacker inventories and then engineers around.
4. **Permission and role structures** — internal role hierarchies ("admin role grants full access to modify user records") that turn the leak into privilege-escalation reconnaissance.

In each case the prompt was doing a job that belongs to an external system: secret storage, policy enforcement, content filtering, or access-control configuration. The application's actual failure "is that the application allows bypassing strong session management and authorization checks by delegating these to the LLM."

## Mitigations: enforce outside the model

The prescribed controls all relocate security state and enforcement out of the prompt ([[owasp-llm-top-10-2025]], LLM07 *Prevention*):

- **Separate sensitive data from system prompts** — no API keys, auth keys, database names, user roles, or permission structure in prompt language; externalize to systems the model cannot read.
- **Avoid relying on system prompts for strict behavior control** — [[prompt-injection]] can alter them; enforce desired behavior in external systems (e.g., harmful-content detection outside the model).
- **Implement guardrails outside the LLM** — an independent system inspecting outputs for compliance, rather than trained-in or prompted-in self-restraint, which "is not a guarantee."
- **Enforce security controls independently from the LLM** — the headline rule: "critical controls such as privilege separation, authorization bounds checks, and similar must not be delegated to the LLM, either through the system prompt or otherwise"; such controls "need to occur in a deterministic, auditable manner, and LLMs are not (currently) conducive to this." Where an agent's tasks require different access levels, use **multiple agents, each configured with the least privileges** needed — privilege separation by agent decomposition rather than by prompt instruction (see [[excessive-agency]], [[tool-use-authorization]]).

Scenario #2 shows why prompt-side defenses compound: an attacker extracts a prompt that prohibits external links and code execution, then uses injection to bypass exactly those instructions, achieving RCE — the leak converts guardrail knowledge into a bypass map ([[owasp-llm-top-10-2025]], LLM07 *Example Attack Scenarios*).

## Relation to pre-AI IAM

The precise analog is **secrets and enforcement logic in client-delivered code**: credentials hard-coded in a JavaScript bundle or mobile binary (CWE-798), and authorization implemented as client-side checks a user can bypass by calling the API directly. The practitioner rule — anything shipped to an environment the user can inspect will be extracted; the server must re-enforce every check — carries over verbatim. "The system prompt is not a security boundary" is the same lesson as "the browser is not a security boundary."

## Why pre-AI IAM is insufficient

Two differences from the client-side-code analog. First, the extraction channel is **conversational and unpatched-able**: no obfuscation or minification applies, and the model itself can be induced to paraphrase its instructions, so leakage resistance cannot be engineered the way binary hardening can. Second, and more fundamental, the system prompt is the *only native control surface* LLM application developers are given, which creates constant pressure to put policy there — the pattern OWASP is pushing against. Pre-AI architectures had an obvious server side on which to enforce; an agentic architecture must deliberately construct its deterministic enforcement points (downstream authorization, external guardrails, [[human-in-the-loop-authorization|human approval]]) because the default location for logic — the prompt — is readable and rewritable by the adversary.
