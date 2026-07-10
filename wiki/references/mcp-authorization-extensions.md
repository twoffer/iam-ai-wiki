---
title: MCP Authorization Extensions (ext-auth)
category: reference
status: stub
confidence: high
aliases: ["ext-auth", "MCP authorization extensions", "MCP auth extensions"]
enterprise_analogs: []
last_updated: 2026-06-18
sources: ["mcp-authorization-overview"]
related: ["mcp-authorization", "mcp-specification"]
tags: ["mcp", "extensions", "authorization", "reference", "stub"]
---

# MCP Authorization Extensions (ext-auth)

**MCP Authorization Extensions** are additional authorization mechanisms layered on top of the core MCP authorization profile. Per the [[mcp-authorization-overview]] (*MCP Authorization Extensions*), they are:

- **Optional** — implementations may adopt them or not.
- **Additive** — they add capabilities without modifying or breaking core protocol behavior.
- **Composable** — designed to work together without conflicts.
- **Versioned independently** — they follow the core MCP versioning cycle but may version separately.

The catalog of supported extensions lives in the `modelcontextprotocol/ext-auth` repository.

## Link

- Repository: <https://github.com/modelcontextprotocol/ext-auth>

## Notes

> **Status: stub.** The overview only names the extensions mechanism and points to the repository; no specific extension has been ingested. Create dedicated pages per extension as they are sourced.
