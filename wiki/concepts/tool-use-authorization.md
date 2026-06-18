---
title: Tool-Use Authorization
category: concept
status: stub
confidence: medium
aliases: [tool authorization, tool scoping, tool-call authorization, tool permissions]
enterprise_analogs: [OAuth scopes, RBAC/ABAC, RFC 8693 token exchange, fine-grained API authorization]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, scope-selection-strategy, step-up-authorization, delegated-authorization, agentic-identity]
tags: [tool-use, scopes, authorization, agentic, stub]
---

# Tool-Use Authorization

**Tool-use authorization** is the question of *what an agent is permitted to do when it invokes a tool* — which operations, on which resources, under whose authority. In MCP, tools are exposed by MCP servers, and the authority to call them is mediated by the [[mcp-authorization|MCP authorization]] flow: OAuth scopes on the access token gate which server operations the client may perform.

## Touchpoint in the MCP Authorization spec

The overview does not enumerate per-tool permissions, but it provides the substrate: scopes are the unit of tool-use authority, and the [[scope-selection-strategy]] plus [[step-up-authorization]] govern how a client acquires exactly the scopes a given tool call requires — "determined dynamically based on the specific request arguments and context" ([[mcp-authorization-overview]], *Runtime Insufficient Scope Errors*). A `403 insufficient_scope` on a tool invocation is the server telling the client it lacks authorization for that specific operation.

## Relation to pre-AI IAM

Mapping tool operations to OAuth scopes is ordinary **fine-grained API authorization** — scopes, RBAC/ABAC policy at the resource server, and token exchange (RFC 8693) for downstream calls. A practitioner models each tool as a protected operation guarded by a scope/permission check.

## Why pre-AI IAM is insufficient

The novelty is that **the caller is an autonomous, non-deterministic agent** choosing tools and arguments at runtime, potentially under adversarial influence. Authorization must therefore be decided per call against the *actual arguments and context*, not pre-bound at integration time, and must resist an agent being steered into invoking a tool it was never meant to. Per-operation scope challenges ([[step-up-authorization]]) are an early answer; richer tool-level policy, argument-aware authorization, and binding tool calls to user intent are open agentic concerns.

> **Status:** stub. Seeded from the MCP overview's scope/operation model; to be expanded by future tool-authorization sources.
