---
title: MCP Security Best Practices (guide)
category: reference
status: stub
confidence: high
aliases: [MCP Security Best Practices, security_best_practices, MCP token passthrough guide]
enterprise_analogs: []
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations]
related: [security-considerations, token-passthrough, confused-deputy, mcp-authorization]
tags: [mcp, security, guide, reference, stub]
---

# MCP Security Best Practices (guide)

A separate MCP documentation page (a tutorial/guide, **not** part of the normative Authorization specification) at <https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices>. The [[mcp-authorization-security-considerations|Security Considerations]] document repeatedly defers to it for the fuller treatment of two threats:

- **Token passthrough** (`#token-passthrough`) — why accepting or forwarding tokens not issued for you is "explicitly forbidden," cited from *Token Audience Binding* and *Access Token Privilege Restriction*. See [[token-passthrough]].
- **Confused deputy problem** (`#confused-deputy-problem`) — the proxy-server/intermediary attack and consent requirements. See [[confused-deputy]].

> **Status: stub.** This guide has **not been ingested** (no copy under `raw/`). The page exists so the spec's repeated cross-references resolve; lint should surface it as a candidate for ingestion. Pull the source and expand if deeper detail is needed than the normative [[security-considerations]] page provides.

## Link

- Guide: <https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices>
