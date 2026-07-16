---
title: OWASP Top 10 for LLM Applications 2025
category: source
status: stable
confidence: high
aliases: ["OWASP LLM Top 10 2025", "LLM01:2025 through LLM10:2025", "OWASP GenAI Top 10 2025"]
enterprise_analogs: ["OWASP Top 10 (web application)", "OWASP API Security Top 10", "least privilege", "complete mediation (Saltzer–Schroeder)"]
last_updated: 2026-07-14
sources: ["raw/owasp-llm-top-10-2025.pdf"]
related: ["owasp-llm-top-10", "owasp-genai-security-project", "prompt-injection", "excessive-agency", "system-prompt-leakage", "vector-store-access-control", "tool-use-authorization", "human-in-the-loop-authorization", "confused-deputy", "delegated-authorization", "machine-identity", "token-theft", "scope-selection-strategy"]
tags: ["owasp", "llm", "threat-model", "top-10", "source"]
---

# OWASP Top 10 for LLM Applications 2025

Summary of `raw/owasp-llm-top-10-2025.pdf` (45 pages, Version 2025, released 2024-11-18 by the [[owasp-genai-security-project|OWASP GenAI Security Project]]; provenance recorded in the sidecar `raw/owasp-llm-top-10-2025.md`, pulled 2026-07-14; CC BY-SA 4.0). The document is the third revision of the community-driven risk list for LLM applications (v1.0 2023-08-01, v1.1 2023-10-16); the series itself is described at [[owasp-llm-top-10]]. Per the wiki's confidence tiers, OWASP LLM Top 10 claims are **high** confidence.

Each of the ten entries follows the same shape: description, common examples of vulnerability/risk, prevention and mitigation strategies, example attack scenarios, reference links, and (for most entries) related frameworks and taxonomies (MITRE ATLAS, NIST). The 2025 revision is notable for this wiki because the project leads explicitly reframed several entries around **agentic architectures**: "Excessive Agency has been expanded, given the increased use of agentic architectures that can give the LLM more autonomy … unchecked permissions can lead to unintended or risky actions" (*Letter from the Project Leads*). System Prompt Leakage (LLM07) and Vector and Embedding Weaknesses (LLM08) are new in 2025; Unbounded Consumption (LLM10) generalizes the old Denial of Service entry.

## Coverage map

Only part of the document is in scope for this wiki (identity, authorization, and delegation for AI agents). Where each entry lives:

| Entry | Wiki treatment | Scope |
| --- | --- | --- |
| LLM01 Prompt Injection | [[prompt-injection]] | In scope (injection as authorization bypass) |
| LLM02 Sensitive Information Disclosure | this page only | Mostly out of scope; least-privilege and injection-bypass points noted below |
| LLM03 Supply Chain | not covered | Out of scope (model/dataset provenance, LoRA tampering) |
| LLM04 Data and Model Poisoning | this page only | Out of scope except the auth-bypass backdoor note below |
| LLM05 Improper Output Handling | this page only; cross-cut on [[tool-use-authorization]] | Slice in scope (output as untrusted input to privileged downstream systems) |
| LLM06 Excessive Agency | [[excessive-agency]] | Fully in scope |
| LLM07 System Prompt Leakage | [[system-prompt-leakage]] | Fully in scope |
| LLM08 Vector and Embedding Weaknesses | [[vector-store-access-control]] | Access-control slice in scope; embedding inversion and RAG quality out of scope |
| LLM09 Misinformation | not covered | Out of scope |
| LLM10 Unbounded Consumption | this page only | Out of scope (DoS, denial of wallet, model extraction) |

## Key claims by entry (in-scope material)

### LLM01: Prompt Injection

A prompt injection vulnerability occurs when prompts "alter the LLM's behavior or output in unintended ways"; inputs need not be human-visible, only model-parsed. The entry distinguishes **direct** injection (the user's own input alters behavior, intentionally or not) from **indirect** injection (the model ingests external content — websites, files — carrying instructions), and distinguishes prompt injection from **jailbreaking** (a form of injection aimed at making the model disregard its safety protocols entirely). Impact "largely depend[s] on … the agency with which the model is architected"; the enumerated outcomes include the authorization-relevant trio of *providing unauthorized access to functions available to the LLM*, *executing arbitrary commands in connected systems*, and *disclosure of sensitive information* (*Description*, *Types*).

The entry concedes there may be no foolproof prevention "given the stochastic influence at the heart of the way models work." Its seven mitigations split cleanly into model-side conditioning (constrain behavior via system prompt, define/validate output formats, input/output filtering) and **authorization-grade controls outside the model**: (4) *enforce privilege control and least privilege access* — the application holds its own API tokens for extensible functionality and handles those functions "in code rather than providing them to the model," restricting the model's privileges to the minimum necessary; (5) *require human approval for high-risk actions* ([[human-in-the-loop-authorization]]); (6) *segregate and identify external content*, limiting untrusted content's influence; (7) *adversarial testing that treats the model as an untrusted user*, testing trust boundaries and access controls (*Prevention*). Attack scenarios include a support chatbot steered into querying private data stores (privilege escalation), webpage-summarization exfiltration via injected image URLs, CVE-2024-5184 (prompt injection in an LLM email assistant), RAG-document manipulation, payload splitting across a résumé, multimodal injection hidden in images, adversarial suffixes, and multilingual/Base64/emoji obfuscation (*Example Attack Scenarios*). MITRE ATLAS mappings: AML.T0051.000/.001 (direct/indirect LLM prompt injection), AML.T0054 (jailbreak).

### LLM02: Sensitive Information Disclosure (in-scope slice)

Mostly a data-privacy entry (PII in training data, sanitization, differential privacy, federated learning) and out of scope. Two points touch this wiki: mitigation via **strict least-privilege access controls** on what data the model and its runtime orchestration can reach (*Access Controls*), and the caution that system-prompt-level restrictions on what the model may return "may not always be honored and could be bypassed via prompt injection or other methods" — restating that model-side instructions are not an authorization control ([[prompt-injection]], [[system-prompt-leakage]]). Attack scenario #2 is *targeted prompt injection* to bypass input filters and extract sensitive information.

### LLM04: Data and Model Poisoning (in-scope note only)

Training-pipeline integrity is out of scope, but scenario #5 names the auth-relevant consequence: a poisoning-installed backdoor trigger "could leave you open to authentication bypass, data exfiltration or hidden command execution" — a compromised-model path to authorization failure that no token-level control observes. The entry also frames trigger-conditioned backdoors as turning the model into a "sleeper agent."

### LLM05: Improper Output Handling (in-scope slice)

Insufficient validation of LLM outputs before they reach downstream components. The IAM-relevant framing: because generated content is steerable by prompt input, passing it downstream unvalidated "is similar to providing users indirect access to additional functionality" — exploitation yields XSS/CSRF in browsers and SSRF, **privilege escalation, or remote code execution** on backends. Named impact amplifiers include the application granting the LLM "privileges beyond what is intended for end users, enabling escalation of privileges" and vulnerability to indirect prompt injection. The first mitigation is the zero-trust rule: "treat the model as any other user," applying proper input validation on model-to-backend responses (*Description*, *Prevention*). This is the output-side complement of [[tool-use-authorization]] and of [[excessive-agency]] (which the entry distinguishes as concerning damaging *actions*, versus scrutiny of *outputs*).

### LLM06: Excessive Agency

The core agentic-authorization entry; full treatment at [[excessive-agency]]. An LLM-based system is granted agency — "the ability to call functions or interface with other systems via extensions" (vendor terms: tools, skills, plugins) — and agent-based systems may let the LLM itself choose which extension to invoke. Excessive Agency "enables damaging actions to be performed in response to unexpected, ambiguous or manipulated outputs from an LLM, regardless of what is causing the LLM to malfunction" — triggers include hallucination, direct/indirect [[prompt-injection]], a compromised extension, or a malicious/compromised peer agent in multi-agent systems. Root causes: **excessive functionality**, **excessive permissions**, **excessive autonomy** (*Description*).

Risk examples are concretely IAM-shaped: an extension whose downstream identity holds UPDATE/INSERT/DELETE when only SELECT is needed; an extension "designed to perform operations in the context of an individual user" that instead connects "with a generic high-privileged identity" with access to all users' files (*Common Examples*). Mitigations: minimize extensions and their functionality; avoid open-ended extensions (run-a-shell-command, fetch-a-URL) in favor of granular ones; minimize extension permissions on downstream systems; **execute extensions in the user's context** (user-authenticated OAuth sessions with minimum scope — [[delegated-authorization]]); **require user approval** for high-impact actions ([[human-in-the-loop-authorization]]); **complete mediation** — "implement authorization in downstream systems rather than relying on an LLM to decide if an action is allowed"; sanitize inputs/outputs per ASVS. Logging and rate limiting are named as damage limiters, not preventers (*Prevention*). The worked scenario — an email-assistant [[confused-deputy]] steered by indirect injection into forwarding sensitive mail — decomposes the fix into eliminating excessive functionality, permissions (read-only OAuth scope), and autonomy (human review before send) (*Example Attack Scenarios*).

### LLM07: System Prompt Leakage

New in 2025; full treatment at [[system-prompt-leakage]]. The entry's central claim is negative and normative: "the system prompt should not be considered a secret, nor should it be used as a security control." Discovery of the prompt's wording is not the real risk; the risk lies in the *underlying elements* — credentials stored where they don't belong, "bypassing strong session management and authorization checks by delegating these to the LLM," improper privilege separation (*Description*). Risk examples: prompts exposing API keys/credentials/connection strings, internal business rules (transaction limits an attacker then works around), filtering criteria, and internal role/permission structures useful for privilege-escalation reconnaissance (*Common Examples*). Mitigations: externalize sensitive data from prompts entirely; avoid relying on system prompts for behavior control (injection can alter them); implement guardrails *outside* the LLM; and the headline rule — critical controls such as "privilege separation, authorization bounds checks" **must not be delegated to the LLM**, "need to occur in a deterministic, auditable manner," and where an agent's tasks require different access levels, use **multiple agents, each configured with the least privileges needed** (*Prevention*). Scenario #1 is credential theft from a leaked prompt ([[token-theft]]); scenario #2 is prompt extraction followed by injection to bypass the discovered guardrails.

### LLM08: Vector and Embedding Weaknesses (in-scope slice)

New in 2025, addressing RAG; the access-control slice is treated at [[vector-store-access-control]]. In-scope risks: **unauthorized access and data leakage** from "inadequate or misaligned access controls" on embeddings containing sensitive information, and **cross-context information leaks in multi-tenant environments** where user or application classes share one vector database and "embeddings from one group might be inadvertently retrieved in response to queries from another group's LLM" (*Common Examples*, scenario #2). The first mitigation is squarely IAM: "implement fine-grained access controls and permission-aware vector and embedding stores," with "strict logical and access partitioning of datasets in the vector database"; supporting controls are data validation/source authentication for knowledge sources and **detailed immutable logs of retrieval activities** (*Prevention*). Scenario #1 (hidden résumé text steering a RAG screening system) is indirect [[prompt-injection]] delivered through the retrieval corpus; the reference list includes the ConfusedPilot attack and "Confused Deputy Risks in RAG-based LLMs" ([[confused-deputy]]). Embedding inversion, federation knowledge conflicts, and RAG behavior alteration are out of scope here.

## Out-of-scope inventory

For lint completeness: **LLM03 Supply Chain** (third-party model/dataset/LoRA-adapter provenance, PoisonGPT, WizardLM impersonation, CloudBorne/CloudJacking, model signing and AI BOMs — closest to this wiki is a passing mention of vendor attestation APIs for on-device models); **LLM09 Misinformation** (hallucination, overreliance, package-hallucination squatting); **LLM10 Unbounded Consumption** (variable-length input floods, denial of wallet, model extraction via API; its mitigation list includes ordinary RBAC/least-privilege on model repositories and MLOps deployment governance, but nothing agent-specific). The bulk of LLM02 (training-data sanitization, differential privacy) and LLM04 (poisoning countermeasures, data version control) is likewise general AI safety/training methodology. Appendix 1 supplies an architecture/threat-model diagram locating each entry on a reference LLM application topology (client → application services → LLM production services, agents, tools, storage, external data sources) with explicit trust boundaries.

## Relation to pre-AI IAM

The document repeatedly lands on classical security principles and says so: least privilege and complete mediation (LLM06's mitigations are nearly verbatim Saltzer–Schroeder), zero trust ("treat the model as any other user," LLM05), secrets-don't-belong-in-client-visible-code (LLM07), and multi-tenant data partitioning (LLM08). Its "Related Frameworks" sections map entries onto MITRE ATLAS techniques the way web-app findings map onto CWE. A practitioner can read LLM06/LLM07 as an over-permissioned-service-account audit and a hard-coded-credentials audit with the LLM substituted for the client application.

## Why pre-AI IAM is insufficient

The through-line the document adds: the component holding delegated authority is now a **stochastic interpreter of untrusted content**. LLM01 states there may be no foolproof injection prevention; LLM07 concludes that authorization must therefore be enforced deterministically *outside* the model; LLM06 concludes that the blast radius of a manipulated model must be pre-bounded by functionality, permission, and autonomy minimization plus human approval — "regardless of what is causing the LLM to malfunction." That inversion — assume the policy-enforcement point inside the model can be subverted, and design the surrounding IAM so subversion doesn't matter — is the document's main contribution to this wiki, and it has no pre-AI analog because no pre-AI deputy executed natural-language instructions from arbitrary data sources.

## Link

- Landing page: <https://genai.owasp.org/resource/owasp-top-10-for-llm-applications-2025/>
- Raw document: `raw/owasp-llm-top-10-2025.pdf` (provenance sidecar: `raw/owasp-llm-top-10-2025.md`)
