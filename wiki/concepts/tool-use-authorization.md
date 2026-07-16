---
title: Tool-Use Authorization
category: concept
status: evolving
confidence: high
aliases: ["tool authorization", "tool scoping", "tool-call authorization", "tool permissions", "extension permissions"]
enterprise_analogs: ["OAuth scopes", "RBAC/ABAC", "RFC 8693 token exchange", "fine-grained API authorization", "complete mediation (Saltzer–Schroeder 1975)"]
last_updated: 2026-07-14
sources: ["mcp-authorization-overview", "mcp-security-best-practices", "owasp-llm-top-10-2025"]
related: ["mcp-authorization", "scope-selection-strategy", "step-up-authorization", "delegated-authorization", "excessive-agency", "agentic-identity", "session-hijacking", "prompt-injection", "system-prompt-leakage"]
tags: ["tool-use", "scopes", "authorization", "agentic"]
---

# Tool-Use Authorization

**Tool-use authorization** is the question of *what an agent is permitted to do when it invokes a tool* — which operations, on which resources, under whose authority. In MCP, tools are exposed by MCP servers, and the authority to call them is mediated by the [[mcp-authorization|MCP authorization]] flow: OAuth scopes on the access token gate which server operations the client may perform. OWASP addresses the same question from the application side, where tools appear as *extensions* (also "plugins" or "skills") whose functionality and downstream permissions must be minimized ([[excessive-agency|LLM06 Excessive Agency]]).

## Touchpoint in the MCP Authorization spec

The overview does not enumerate per-tool permissions, but it provides the substrate: scopes are the unit of tool-use authority, and the [[scope-selection-strategy]] plus [[step-up-authorization]] govern how a client acquires exactly the scopes a given tool call requires — "determined dynamically based on the specific request arguments and context" ([[mcp-authorization-overview]], *Runtime Insufficient Scope Errors*). A `403 insufficient_scope` on a tool invocation is the server telling the client it lacks authorization for that specific operation.

The [[mcp-security-best-practices|Security Best Practices]] guide adds two edges. First, **scopes are necessary but not sufficient**: treating claimed scopes in a token as authorization "without server-side authorization logic" is a named common mistake — the resource server still decides each operation (*Scope Minimization → Common Mistakes*). Second, **tool inventory integrity is part of tool-use authorization**: in the [[session-hijacking|session-hijack prompt-injection]] scenario, a forged `notifications/tools/list_changed` can leave a client "with tools that they were not aware were enabled" — the *set* of invocable tools, not just permission to call them, is attacker-influencable state that must be protected.

## The OWASP design rules (LLM06)

[[owasp-llm-top-10-2025|OWASP LLM06]] supplies the design discipline for the tool layer itself, decomposing over-grant into excessive functionality, permissions, and autonomy (see [[excessive-agency]]) and prescribing per-dimension minimization (*Prevention*):

- **Minimize the tool set.** Offer the agent only the extensions its task needs; remove trialled-and-dropped plugins that remain invocable.
- **Minimize each tool's functionality.** A mailbox-summarizing extension needs read; it should not also implement send and delete. Avoid **open-ended tools** (run a shell command, fetch a URL) in favor of granular ones — a file-writing tool rather than a shell tool, because an open-ended tool's undesirable-action scope is "any other command."
- **Minimize downstream permissions.** The identity a tool presents to downstream systems carries only what the intended operation needs — a product-recommendation agent's database identity gets read on the `products` table, not UPDATE/INSERT/DELETE — "enforced by applying appropriate database permissions for the identity that the LLM extension uses to connect."
- **Execute in the user's context.** Per-user operations run under the requesting user's authority with minimum privileges (e.g., user-authenticated OAuth with the minimum scope), never a generic high-privileged identity — [[delegated-authorization]] rather than ambient [[machine-identity]].
- **Complete mediation.** "Implement authorization in downstream systems rather than relying on an LLM to decide if an action is allowed"; every tool-initiated request is validated against security policy at the resource. This converges exactly with the MCP guide's scopes-without-server-side-authz mistake, and with [[system-prompt-leakage|LLM07]]'s rule that authorization bounds checks must not be delegated to the model.

For high-impact operations, per-call [[human-in-the-loop-authorization|human approval]] sits on top of the permission model; logging and rate limiting are damage limiters, not authorization.

## Relation to pre-AI IAM

Mapping tool operations to OAuth scopes is ordinary **fine-grained API authorization** — scopes, RBAC/ABAC policy at the resource server, and token exchange (RFC 8693) for downstream calls. A practitioner models each tool as a protected operation guarded by a scope/permission check, and OWASP's complete-mediation rule is the classic Saltzer–Schroeder principle restated for the LLM boundary.

## Why pre-AI IAM is insufficient

The novelty is that **the caller is an autonomous, non-deterministic agent** choosing tools and arguments at runtime, potentially under adversarial influence — OWASP's excessive-agency triggers (hallucination, [[prompt-injection|injection]], compromised extensions, malicious peer agents) are all ways the tool-selecting logic itself goes wrong. Authorization must therefore be decided per call against the *actual arguments and context*, not pre-bound at integration time, and the tool layer must be designed so that a steered agent's reachable actions are already minimal. Per-operation scope challenges ([[step-up-authorization]]) and LLM06's minimization discipline are early answers; richer argument-aware policy and binding tool calls to user intent remain open agentic concerns.
