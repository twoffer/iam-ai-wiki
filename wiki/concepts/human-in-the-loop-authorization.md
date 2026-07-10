---
title: Human-in-the-Loop Authorization
category: concept
status: evolving
confidence: high
aliases: [human-in-the-loop, HITL authorization, user consent, interactive consent, consent UI]
enterprise_analogs: [OAuth 2.1 authorization/consent endpoint, OIDC `prompt=consent`, OAuth incremental consent, OS elevation prompts (UAC, Gatekeeper)]
last_updated: 2026-07-08
sources: [mcp-authorization-overview, mcp-security-best-practices]
related: [delegated-authorization, step-up-authorization, scope-selection-strategy, mcp-authorization, agentic-identity, confused-deputy, local-mcp-server-security, prompt-injection]
tags: [consent, human-in-the-loop, authorization]
---

# Human-in-the-Loop Authorization

**Human-in-the-loop (HITL) authorization** is the requirement that a human explicitly approve the authority an agent acquires or the actions it takes. In OAuth terms it is the **user consent step**: the point where the resource owner sees what is being requested and grants or denies it.

## Touchpoint in the MCP Authorization spec

In the [[mcp-authorization|MCP flow]], the human appears at the browser leg: the authorization server "interact[s] with the user (if necessary)" and the diagram's "User authorizes" step is where consent is captured ([[mcp-authorization-overview]], *Roles* / *Authorization Flow Steps*). The [[scope-selection-strategy]] is explicitly designed around this moment — a general-purpose client requests the available scopes and lets "the authorization server and end-user determine appropriate permissions during the consent process." [[step-up-authorization]] re-invokes the human when new scopes are needed.

## The two consent surfaces in the Security Best Practices guide

The [[mcp-security-best-practices|Security Best Practices]] guide turns consent from a UX nicety into a normative security control at two distinct points:

**Proxy-server per-client consent.** An MCP proxy fronting a third-party API with a static client ID **MUST** run its own consent page for each dynamically registered downstream client, *before* forwarding to the third-party AS — the defense against the [[confused-deputy|consent-cookie confused-deputy attack]]. The page itself carries MUSTs: clearly identify the requesting client by name, display the specific third-party scopes requested, show the registered `redirect_uri` where tokens will be sent, implement CSRF protection, and prevent clickjacking via `frame-ancestors` CSP or `X-Frame-Options: DENY`. Consent decisions are stored per user per `client_id`, and the OAuth `state` for the onward flow may be established **only after** the user approves — a mis-sequenced implementation "renders the consent screen ineffective," i.e., consent that is not enforced at the callback is not consent.

**Local-server pre-execution consent.** An MCP client offering one-click [[local-mcp-server-security|local server]] configuration **MUST** obtain consent before executing anything: show the exact command **without truncation** (arguments included), identify it as a potentially dangerous operation that executes code on the user's system, require explicit approval, and allow cancellation — with SHOULD-level guardrails such as highlighting dangerous patterns (`sudo`, `rm -rf`) and warning on access to SSH keys or system directories. Here the human approves not an OAuth scope but a *process execution* — HITL as the last control before arbitrary code runs with the client's privileges.

Both surfaces share a design principle: the dialog must give the human the **material facts of the delegation** (who is asking, what authority, where results flow), and the system must **enforce downstream exactly what was approved**.

## Relation to pre-AI IAM

This is the ordinary OAuth **consent screen** and authorization endpoint, plus patterns like OIDC `prompt=consent` and incremental consent; the local-execution variant is the OS elevation prompt (UAC, Gatekeeper). Practitioners already treat user approval as the trust anchor of delegated access, and the consent-page hardening rules (CSRF, anti-clickjacking, exact enforcement of the approved client/scopes) are standard secure-web-app discipline applied to the consent endpoint.

## Why pre-AI IAM is insufficient

Classic consent is a **one-time, up-front** event for a known app requesting a known scope set. Agentic systems need consent that is **per-operation, runtime, and legible** — the user must understand what an autonomous, possibly [[prompt-injection|injection-influenced]] agent is about to do, often mid-task and for actions the developer did not enumerate in advance. The MCP corpus now codifies pieces of this: per-downstream-client consent at proxies (because the "app" population is open and dynamically registered) and pre-execution consent for local servers (because installing agent tooling is a casual, high-frequency act). What remains open is keeping such consent meaningful under agent autonomy — avoiding rubber-stamping while limiting friction — which scope challenges and consent dialogs only partially address.
