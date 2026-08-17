---
title: GitHub MCP Server
category: entity
status: stub
confidence: medium
aliases: ["github-mcp-server", "GitHub MCP integration", "official GitHub MCP server"]
enterprise_analogs: []
last_updated: 2026-08-17
sources: ["invariant-github-mcp-vulnerability"]
related: ["github-mcp-private-repo-leak", "toxic-agent-flow", "mcp-specification", "tool-use-authorization", "prompt-injection"]
tags: ["mcp-server", "github", "product"]
---

# GitHub MCP Server

The **GitHub MCP server** (github.com/github/github-mcp-server) is GitHub's official [[mcp-specification|MCP]] integration exposing GitHub operations — reading issues, repositories, and pull requests, creating PRs, and more — as tools an MCP client can call under a user's GitHub credentials. The writeup describes it as widely used ("14k stars on GitHub").

It is the trusted tool through which the [[github-mcp-private-repo-leak|GitHub MCP private-repo leak]] flows: the server itself is not vulnerable — Invariant explicitly states the exploit is "not a flaw in the GitHub MCP server code itself" — but its tools give an agent, under a single over-broad token, the ability to read private repositories and create public pull requests in one session, which a [[prompt-injection|prompt injection]] can chain into an exfiltration [[toxic-agent-flow]]. The core lesson is that the fix must come from the agent system's authorization scoping ([[tool-use-authorization]]), not from the server.

Stub — expand if a source documents the server's tool inventory or auth model directly.
