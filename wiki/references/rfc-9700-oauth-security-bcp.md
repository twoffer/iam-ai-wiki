---
title: RFC 9700 — Best Current Practice for OAuth 2.0 Security
category: reference
status: stub
confidence: high
aliases: ["RFC 9700", "OAuth 2.0 Security BCP", "OAuth Security Best Current Practice", "OAuth 2.0 security best practices"]
enterprise_analogs: []
last_updated: 2026-07-08
sources: ["mcp-security-best-practices"]
related: ["oauth-2-1", "security-considerations", "mcp-security-best-practices", "confused-deputy", "open-redirection", "token-theft", "authorization-server-mix-up"]
tags: ["oauth", "security", "bcp", "ietf", "reference", "stub"]
---

# RFC 9700 — Best Current Practice for OAuth 2.0 Security

**RFC 9700** is the IETF's consolidated security guidance for OAuth 2.0 deployments — the "OAuth 2.0 Security Best Current Practice" — updating and extending the original threat model (RFC 6819) with attacks and mitigations learned since: mix-up defenses, redirect-URI hardening, token-replay prevention, and sender-constraining guidance. Much of it was folded into [[oauth-2-1|OAuth 2.1]].

## Role in the MCP corpus

The [[mcp-security-best-practices|MCP Security Best Practices]] guide instructs that it "should be read alongside" the MCP Authorization specification and RFC 9700 — positioning the BCP as the pre-AI baseline the MCP-specific guidance builds on. Wiki pages routinely cite it as the enterprise ancestor of MCP's mandatory mitigations (see [[security-considerations]], [[confused-deputy]], [[open-redirection]], [[token-theft]], [[authorization-server-mix-up]]).

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc9700>

> **Status: stub.** The RFC itself has not been ingested (no copy under `raw/`); this page exists so the wiki's many RFC 9700 mentions resolve. Ingesting it would let the per-threat pages cite BCP section numbers directly.
