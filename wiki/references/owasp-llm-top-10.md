---
title: OWASP Top 10 for LLM Applications
category: reference
status: evolving
confidence: high
aliases: ["OWASP LLM Top 10", "LLM Top 10", "OWASP Top 10 for Large Language Model Applications", "GenAI Top 10"]
enterprise_analogs: ["OWASP Top 10 (web application)", "OWASP API Security Top 10", "CWE/SANS Top 25"]
last_updated: 2026-07-14
sources: ["owasp-llm-top-10-2025"]
related: ["owasp-genai-security-project", "owasp-llm-top-10-2025", "prompt-injection", "excessive-agency", "system-prompt-leakage", "vector-store-access-control"]
tags: ["owasp", "llm", "top-10", "reference", "threat-model"]
---

# OWASP Top 10 for LLM Applications

The **OWASP Top 10 for LLM Applications** is the community-driven risk list for applications built on large language models, published by the [[owasp-genai-security-project|OWASP GenAI Security Project]]. Started in 2023 "to highlight and address security issues specific to AI applications," it is produced by contributor brainstorming, voting, and practitioner feedback, in the tradition of the web-application OWASP Top 10. It is licensed CC BY-SA 4.0. Per this wiki's confidence tiers, it is a **high**-confidence source.

## Revisions

| Version | Date | Notes |
| --- | --- | --- |
| 1.0 | 2023-08-01 | Initial release |
| 1.1 | 2023-10-16 | Revision |
| 2025 | 2024-11-18 | Current; ingested as [[owasp-llm-top-10-2025]] |

Changes in the 2025 version relevant to this wiki: **Excessive Agency** (LLM06) expanded for agentic architectures ("unchecked permissions can lead to unintended or risky actions, making this entry more critical than ever"); **System Prompt Leakage** (LLM07) added after real-world exploits broke the assumption that prompts are securely isolated; **Vector and Embedding Weaknesses** (LLM08) added to cover RAG; *Unbounded Consumption* (LLM10) generalized from the earlier Denial of Service entry.

## The 2025 list

LLM01 [[prompt-injection|Prompt Injection]] · LLM02 Sensitive Information Disclosure · LLM03 Supply Chain · LLM04 Data and Model Poisoning · LLM05 Improper Output Handling · LLM06 [[excessive-agency|Excessive Agency]] · LLM07 [[system-prompt-leakage|System Prompt Leakage]] · LLM08 Vector and Embedding Weaknesses ([[vector-store-access-control|access-control slice]]) · LLM09 Misinformation · LLM10 Unbounded Consumption. Entries follow a fixed structure (description, examples, prevention/mitigation, attack scenarios, references, MITRE ATLAS/NIST mappings); see [[owasp-llm-top-10-2025]] for the claims and this wiki's scope filter.

## Link

- <https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/>
