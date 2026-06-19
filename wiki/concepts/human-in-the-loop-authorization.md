---
title: Human-in-the-Loop Authorization
category: concept
status: stub
confidence: medium
aliases: [human-in-the-loop, HITL authorization, user consent, interactive consent]
enterprise_analogs: [OAuth 2.1 authorization/consent endpoint, OIDC `prompt=consent`, OAuth incremental consent]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [delegated-authorization, step-up-authorization, scope-selection-strategy, mcp-authorization, agentic-identity]
tags: [consent, human-in-the-loop, authorization, stub]
---

# Human-in-the-Loop Authorization

**Human-in-the-loop (HITL) authorization** is the requirement that a human explicitly approve the authority an agent acquires or the actions it takes. In OAuth terms it is the **user consent step**: the point where the resource owner sees what is being requested and grants or denies it.

## Touchpoint in the MCP Authorization spec

In the [[mcp-authorization|MCP flow]], the human appears at the browser leg: the authorization server "interact[s] with the user (if necessary)" and the diagram's "User authorizes" step is where consent is captured ([[mcp-authorization-overview]], *Roles* / *Authorization Flow Steps*). The [[scope-selection-strategy]] is explicitly designed around this moment — a general-purpose client requests the available scopes and lets "the authorization server and end-user determine appropriate permissions during the consent process." [[step-up-authorization]] re-invokes the human when new scopes are needed.

## Relation to pre-AI IAM

This is the ordinary OAuth **consent screen** and authorization endpoint, plus patterns like OIDC `prompt=consent` and incremental consent. Practitioners already treat user approval as the trust anchor of delegated access.

## Why pre-AI IAM is insufficient

Classic consent is a **one-time, up-front** event for a known app requesting a known scope set. Agentic systems need consent that is **per-operation, runtime, and legible** — the user must understand what an autonomous, possibly injection-influenced agent is about to do, often mid-task and for actions the developer did not enumerate in advance. Designing consent that stays meaningful under agent autonomy (avoiding rubber-stamping while limiting friction) is an open problem this spec only partially addresses via runtime scope challenges.

> **Status:** stub. Seeded from the MCP overview's consent and scope-selection mentions; to be expanded by future HITL / consent-UX sources.
