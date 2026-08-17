---
title: Invariant Labs
category: entity
status: stub
confidence: medium
aliases: ["Invariant", "Invariant Labs AI"]
enterprise_analogs: []
last_updated: 2026-08-17
sources: ["invariant-github-mcp-vulnerability"]
related: ["toxic-agent-flow", "github-mcp-private-repo-leak", "prompt-injection", "tool-poisoning", "tool-use-authorization"]
tags: ["vendor", "security-research", "mcp", "guardrails"]
---

# Invariant Labs

**Invariant Labs** is an AI-security company focused on agentic systems and MCP. Its public research coined the term [[toxic-agent-flow|toxic agent flow]] and reported the [[github-mcp-private-repo-leak|GitHub MCP private-repo leak]] (2025) and earlier MCP [[tool-poisoning]] attacks. It sells agent-security products referenced across its writeups:

- **Guardrails** — a runtime policy engine enforcing context-aware rules over an agent's tool-call flow (e.g. the "one repo per session" policy proposed against the GitHub MCP leak).
- **MCP-scan** — a scanner for auditing agent↔MCP interactions, including a proxy mode that inspects MCP traffic in real time without modifying the agent stack.
- **Explorer / Guardrails Playground** — trace inspection and policy-testing tooling.

As a vendor, Invariant's claims about its own products' effectiveness are self-interested; its incident research is cited in this wiki at medium confidence per the source tiers. Authors of the GitHub MCP writeup: Marco Milanta and Luca Beurer-Kellner.

Stub — expand if further Invariant research is ingested.
