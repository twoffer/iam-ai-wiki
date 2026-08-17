---
title: Toxic Agent Flow
category: concept
status: evolving
confidence: medium
aliases: ["toxic agent flow", "toxic flow", "toxic agent flows"]
enterprise_analogs: ["confused deputy (Hardy 1988)", "CSRF", "gadget chain / exploit chain", "data-exfiltration kill chain"]
last_updated: 2026-08-17
sources: ["invariant-github-mcp-vulnerability"]
related: ["prompt-injection", "confused-deputy", "excessive-agency", "tool-use-authorization", "github-mcp-private-repo-leak", "invariant-labs", "tool-poisoning", "human-in-the-loop-authorization"]
tags: ["agentic", "prompt-injection", "threat-model", "exfiltration"]
---

# Toxic Agent Flow

A **toxic agent flow** is the term Invariant Labs uses for an indirect [[prompt-injection]] that triggers a malicious *sequence* of tool calls, steering an agent into unintended actions such as exfiltrating data or executing attacker-chosen code ([[invariant-github-mcp-vulnerability]]). The emphasis is on the *flow*: the harm is not any single tool call but the chain — read untrusted content, then combine otherwise-legitimate tool calls into an exfiltration or destructive path the user never intended.

## Defining properties

- **Trusted tools, untrusted data.** A toxic agent flow requires no compromised or malicious tool. It arises when an agent using fully trusted tools ingests untrusted content through one of them — the flow is a property of the *system's* data and authority topology, not of any component. This distinguishes it from [[tool-poisoning]], where the tool definition itself is the attack vector.
- **Injection is the trigger, agency is the payload.** The injection only redirects control flow; the damage depends on the authority the agent already holds across its tool set — an instance of [[excessive-agency]]. A flow becomes "toxic" precisely when a steered agent can chain its standing authority into an outcome that crosses a trust boundary.
- **The chain crosses a boundary the individual calls do not.** Each tool call in the sequence may be independently authorized; the toxicity is emergent. In the [[github-mcp-private-repo-leak|GitHub MCP leak]], listing public issues, reading a private repo, and opening a public PR are each within the token's grant — their *composition* moves private data to a public surface.

## Detection and mitigation

Invariant frames toxic agent flows as something to discover proactively — its writeup reports finding the GitHub MCP flow with an automated security analyzer rather than manual review — and to bound at the agent-system level rather than the model level, since even a highly aligned model and off-the-shelf injection detectors did not stop the demonstrated attack. The recommended controls are authority-bounding and flow-aware: runtime policy over the tool-call sequence (e.g. Invariant Guardrails' "one repo per session" rule that trips when consecutive tool calls target different resources — see [[invariant-labs]], [[tool-use-authorization]]) and continuous monitoring of agent↔tool traffic. Per-action [[human-in-the-loop-authorization]] is a partial control that consent fatigue can erode.

## Relation to pre-AI IAM

The closest analogs are the [[confused-deputy]] (a privileged party induced to misuse standing authority) and the exploit/gadget *chain*, where individually benign primitives are composed into a capability none provides alone. Practitioners already reason about data-exfiltration kill chains and about scoping credentials so no single identity can both read sensitive data and reach an external sink.

## Why pre-AI IAM is insufficient

In pre-AI systems the "gadgets" are chained by attacker-written code exploiting a bug; here they are chained by a probabilistic agent following instructions embedded in data it was legitimately asked to read, with no reliable control/data separation to close the path. The dangerous composition is not a fixed code path to patch but any sequence a steered agent can be talked into, drawn from its whole authorized tool set. Defense therefore shifts from preventing the trigger to constraining the *flow* — per-session, argument-aware limits on which combinations of authorized actions one agent run may perform — a policy surface pre-AI IAM never had to express.
