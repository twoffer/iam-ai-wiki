---
title: Model Context Protocol (MCP) Specification
category: reference
status: evolving
confidence: high
aliases: ["MCP", "Model Context Protocol", "modelcontextprotocol.io", "MCP spec"]
enterprise_analogs: []
last_updated: 2026-07-08
sources: ["mcp-authorization-overview", "mcp-authorization-server-discovery"]
related: ["mcp-authorization", "anthropic", "mcp-authorization-extensions", "authorization-server-discovery", "client-registration", "security-considerations", "mcp-security-best-practices"]
tags: ["mcp", "protocol", "spec", "reference"]
---

# Model Context Protocol (MCP) Specification

The **Model Context Protocol (MCP)** is an open protocol standardizing how applications provide context and tools to LLM-based clients. It defines clients (hosts/agents), servers (which expose tools, resources, and prompts), and the transports between them (STDIO and HTTP-based). Originated by [[anthropic|Anthropic]] and developed as an open specification at <https://modelcontextprotocol.io>.

## Authorization within MCP

Authorization is an **optional**, transport-level part of the spec, defined for HTTP-based transports as a profile of [[oauth-2-1|OAuth 2.1]]. The authorization specification spans four documents, all ingested:

- [[mcp-authorization]] (overview) — see [[mcp-authorization-overview]].
- [[authorization-server-discovery]] — see [[mcp-authorization-server-discovery]].
- [[client-registration]] — see [[mcp-authorization-client-registration]].
- [[security-considerations]] — see [[mcp-authorization-security-considerations]].

The documentation site also carries a non-spec [[mcp-security-best-practices|Security Best Practices]] guide (tutorials section) that the Security Considerations document defers to; it is ingested as well. Optional, additive [[mcp-authorization-extensions|authorization extensions]] are maintained separately.

## Link

- Site: <https://modelcontextprotocol.io>
- Authorization (draft): <https://modelcontextprotocol.io/specification/draft/basic/authorization>

## Notes

This wiki currently covers the **draft** revision's authorization chapter plus the security best-practices tutorial. Other parts of the MCP spec (lifecycle, tools, resources, prompts, transports) are out of the ingested scope so far.
