---
title: Confused Deputy
category: concept
status: stable
confidence: high
aliases: ["confused deputy problem", "confused deputy attack", "deputy confusion", "MCP proxy server attack"]
enterprise_analogs: ["OAuth 2.0 mix-up attack", "CSRF", "RFC 9700 OAuth 2.0 Security BCP", "ambient authority", "consent-cookie bypass"]
last_updated: 2026-07-14
sources: ["mcp-security-best-practices", "mcp-authorization-security-considerations", "mcp-authorization-overview", "owasp-llm-top-10-2025"]
related: ["token-audience-binding", "token-passthrough", "authorization-server-mix-up", "client-registration", "mcp-authorization", "security-considerations", "agentic-identity", "delegated-authorization", "mcp-security-best-practices", "open-redirection", "human-in-the-loop-authorization", "prompt-injection", "excessive-agency", "vector-store-access-control"]
tags: ["security", "confused-deputy", "threat-model", "core-concept"]
---

# Confused Deputy

A **confused deputy** is a privileged component tricked into misusing its authority on behalf of a less-privileged or malicious party. The deputy is not compromised in the sense of running attacker code; it is *confused* into applying its legitimate credentials to an attacker-chosen request. The term predates AI (Norm Hardy, 1988), but agentic systems make it a central threat: an AI agent is, almost by definition, a deputy that wields delegated authority across many tools and servers.

## In agentic / MCP contexts

An MCP client holds tokens delegated by a user and connects to multiple servers. Two confused-deputy shapes matter:

- **Cross-resource token misuse.** If a token were not [[token-audience-binding|audience-bound]], a malicious server (or an instruction injected into content the agent processes) could induce the client to present a token — minted for server A — to server B, exercising the user's authority where it was never granted. MCP's mandatory audience binding and [[token-passthrough]] prohibition are the structural fix.
- **Authorization-server mix-up.** A client tricked into talking to the wrong AS, or redeeming a code at the wrong endpoint, can leak a code/token to an attacker. MCP's mandatory [[rfc-9207-authorization-server-issuer-identification|`iss` validation]] and discovery rules defend this; see [[authorization-server-mix-up]].

The [[mcp-authorization-overview]] lists "mix-up and confused deputy attacks" among the threats its [[security-considerations]] chapter addresses normatively, and the [[mcp-authorization-security-considerations|Security Considerations]] document gives the canonical MCP instance below.

## The generic LLM-application instance (OWASP)

Outside MCP's protocol frame, the [[owasp-llm-top-10|OWASP LLM Top 10]] supplies the everyday agentic instance: its [[excessive-agency|LLM06 Excessive Agency]] scenario is an email-assistant extension that, steered by an indirect [[prompt-injection|injection]] in an incoming email, scans the user's inbox for sensitive information and forwards it to the attacker — the deputy's legitimate mailbox authority applied to an attacker-chosen request ([[owasp-llm-top-10-2025]], LLM06 *Example Attack Scenarios*; the entry's own reference list includes an "Embrace the Red: Confused Deputy Problem" write-up). The exploitability of the deputy is a direct function of its excessive agency — functionality, permissions, and autonomy beyond the task — which is why OWASP's mitigations (minimization, user-context execution, complete mediation, [[human-in-the-loop-authorization|human approval]]) are all authority-bounding rather than confusion-preventing. The RAG variant, where a shared retrieval pipeline exercises broad read authority on behalf of differently privileged queriers, is covered at [[vector-store-access-control]] (cf. the ConfusedPilot attack referenced by LLM08).

## The proxy-server / consent case

The most concrete MCP confused-deputy scenario is an **MCP proxy server that fronts a third-party API** using a **static (pre-registered) client ID** at the third party's authorization server. Because the third-party AS already has consent on file for that static client, it may **skip its consent screen** on subsequent authorizations. An attacker who can drive a [[client-registration|dynamically registered]] downstream client through the proxy can ride that prior consent and, using a stolen authorization code, obtain tokens **without the user actually consenting** ([[mcp-authorization-security-considerations]], *Confused Deputy Problem*). The [[mcp-security-best-practices|Security Best Practices]] guide supplies the full anatomy, summarized below.

### Vulnerable conditions

The attack requires **all four** conditions simultaneously ([[mcp-security-best-practices]], *Vulnerable Conditions*):

1. The proxy uses a **static client ID** with the third-party AS (which may lack dynamic registration, forcing one shared upstream identity for every downstream client).
2. The proxy lets MCP clients **dynamically register**, each receiving its own `client_id` and self-declared `redirect_uri`.
3. The third-party AS sets a **consent cookie** on the user agent after the first approval, keyed to the static client ID.
4. The proxy does **not** run per-client consent before forwarding to the third-party AS.

### Attack anatomy

A user first authorizes legitimately; the third-party AS drops its consent cookie for the proxy's static client ID. Later, the attacker dynamically registers a malicious client whose `redirect_uri` points at attacker infrastructure and sends the user a crafted authorization link. The user's browser still carries the consent cookie, so the third-party AS **skips the consent screen** and issues its code; the proxy exchanges it, mints an **MCP authorization code**, and — following the malicious registration — redirects it to the attacker's `redirect_uri`. The attacker exchanges that code for MCP tokens and reaches the third-party API as the user, who approved nothing ([[mcp-security-best-practices]], *Attack Description*). The deputy is the proxy: its standing upstream consent was applied to a request the user never sanctioned.

### The mitigation stack

The headline rule — proxies using static client IDs **MUST obtain user consent for each dynamically registered client before forwarding to third-party authorization servers** — decomposes into concrete controls ([[mcp-security-best-practices]], *Mitigation*):

- **Per-client consent storage (MUST).** Keep a per-user registry of approved `client_id` values, check it **before** initiating the third-party flow, and store decisions securely (server-side or in server-specific cookies).
- **Consent UI (MUST).** The proxy-owned consent page names the requesting client, displays the third-party scopes requested, shows the registered `redirect_uri` tokens will be sent to, and carries CSRF protection and anti-clickjacking headers (`frame-ancestors` CSP / `X-Frame-Options: DENY`). See [[human-in-the-loop-authorization]].
- **Consent-cookie hygiene (MUST).** Consent-tracking cookies use the `__Host-` prefix, `Secure`/`HttpOnly`/`SameSite=Lax`, cryptographic signing or server-side sessions, and bind to the specific `client_id` — never a bare "user has consented" flag.
- **Redirect-URI validation (MUST).** Exact string matching against the registered URI — no patterns or wildcards — rejecting any change made without re-registration (see [[open-redirection]]).
- **`state` lifecycle (MUST).** A cryptographically random, single-use, short-lived (~10 min) `state` per authorization request, exactly matched at the callback against the cookie/session copy. Crucially, the `state`-bearing cookie or session is stored **only after the user approves the consent screen** and set immediately before the third-party redirect — setting it earlier "renders the consent screen ineffective," since an attacker could then drive the flow past a screen the user never saw.

The consent-timing point is what makes this a *deputy* problem rather than a plumbing bug: consent approval at the authorization endpoint must be **enforced at the callback endpoint**, or the deputy honors authority the user never granted. This is distinct from the audience-binding defense: it protects the **consent** step rather than the token-audience step. The complementary token-level rule — never relay the inbound token to the upstream, mint a separate one — is the [[token-passthrough]] "two hats" requirement.

## Relation to pre-AI IAM

This is a well-known OAuth threat class. The **OAuth 2.0 mix-up attack**, **CSRF** on the redirect, and **ambient-authority** abuse are all confused-deputy instances, catalogued in the OAuth 2.0 Security Best Current Practice (RFC 9700). The standard mitigations — exact redirect-URI matching, PKCE, `state`, audience restriction, issuer identification — are the same primitives MCP mandates. A practitioner's existing confused-deputy mental model applies directly.

## Why pre-AI IAM is insufficient

The classic mitigations assume a **fixed, human-integrated topology** and a deputy whose behavior is deterministic code. Agentic systems break both:

- The deputy's control flow is **influenced by untrusted natural-language input** ([[prompt-injection]]), so an attacker can steer *which* request the deputy makes, not just intercept it. Authority confusion can be induced through data, not only through protocol manipulation — see [[session-hijacking]] for a sourced infrastructure-level delivery vector.
- The deputy brokers **many dynamically discovered servers**, so the population of potential victims/attackers is open and unbounded.

This is why MCP promotes optional OAuth hardening (audience binding, issuer validation, no token passthrough) to **mandatory**, adds consent-per-dynamic-client for proxy servers, and why prompt-injection-as-authorization-bypass is a first-class agentic concern beyond anything the pre-AI threat model required. The proxy-server consent requirement in particular has no clean pre-AI analog: classic OAuth had no notion of an intermediary that dynamically registers downstream clients on the fly.
