---
title: Tool Poisoning
category: concept
status: stub
confidence: low
aliases: ["tool poisoning", "tool poisoning attack", "MCP tool poisoning"]
enterprise_analogs: ["malicious dependency / supply-chain attack", "trojaned plugin"]
last_updated: 2026-08-17
sources: ["invariant-github-mcp-vulnerability"]
related: ["toxic-agent-flow", "prompt-injection", "tool-use-authorization", "invariant-labs", "github-mcp-private-repo-leak"]
tags: ["mcp", "prompt-injection", "supply-chain", "stub"]
---

# Tool Poisoning

**Tool poisoning** is an MCP attack class in which the MCP *tool itself* is the attack vector — a malicious or compromised tool definition (e.g. injection payloads hidden in a tool's description or schema) manipulates the agent. Invariant's [[github-mcp-private-repo-leak|GitHub MCP writeup]] contrasts it with [[toxic-agent-flow|toxic agent flows]]: tool poisoning requires the tools to be compromised, whereas a toxic agent flow emerges "even with fully trusted tools" via untrusted content the agent reads through them ([[invariant-github-mcp-vulnerability]]).

Stub — only referenced in passing by the Invariant writeup; ingest Invariant's dedicated tool-poisoning post (invariantlabs.ai/blog on MCP tool-poisoning attacks) to substantiate and raise confidence.
