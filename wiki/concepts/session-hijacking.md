---
title: Session Hijacking
category: concept
status: stable
confidence: high
aliases: [session hijack, session ID theft, session hijack prompt injection, session hijack impersonation]
enterprise_analogs: [OWASP session management (hijacking/fixation), HTTP session cookie theft, session-ID entropy requirements, RFC 6750 bearer tokens (credential vs session distinction)]
last_updated: 2026-07-08
sources: [mcp-security-best-practices]
related: [mcp-authorization, token-theft, prompt-injection, tool-use-authorization, security-considerations, confused-deputy, local-mcp-server-security]
tags: [security, session, prompt-injection, threat-model, core-concept]
---

# Session Hijacking

**Session hijacking** is the attack in which an unauthorized party obtains or guesses the session ID a server issued to a client and uses it to act as that client. MCP's streamable HTTP transport is session-based — the server hands the client a session ID at initialization — so the classic web threat carries over directly, with one distinctly agentic twist: hijacking an MCP session can be used not only to *impersonate* the client but to *inject content the client's agent will act on* ([[mcp-security-best-practices]], *Session Hijacking*).

## The two MCP variants

**Session hijack impersonation.** The attacker replays a known/guessed session ID against the MCP server. If the server treats a valid session ID as proof of identity and does not re-verify authorization, the attacker is served as if they were the legitimate client — the standard pre-AI outcome.

**Session hijack prompt injection.** The variant specific to horizontally scaled, stateful deployments. Multiple server instances share an event queue keyed by session ID. The attacker sends a malicious event to instance B using the victim's session ID; instance A later polls the queue for that session and delivers the attacker's payload to the legitimate client as an asynchronous or resumed response; the client acts on it. Two protocol features sharpen this: with [resumable streams](https://modelcontextprotocol.io/specification/latest/basic/transports#resumability-and-redelivery), an attacker can deliberately terminate a request so the victim's client resumes — and receives — the malicious response via its SSE `GET`; and a forged `notifications/tools/list_changed` can alter the tool set the client believes the server offers, leaving the client "with tools that they were not aware were enabled" ([[mcp-security-best-practices]]). The session ID here functions as a **write credential into the agent's input stream** — a [[prompt-injection]] delivery channel and a route to corrupting [[tool-use-authorization|tool inventory]].

## Mitigations

The guide's normative requirements ([[mcp-security-best-practices]], *Mitigation*):

- **Verify every inbound request (MUST).** MCP servers that implement authorization MUST verify all inbound requests — possession of a session ID never substitutes for the access token.
- **Sessions are not authentication (MUST NOT).** MCP servers MUST NOT use sessions for authentication. The session is a continuity mechanism; identity and authority come from the [[mcp-authorization|OAuth token]] on each request.
- **Non-deterministic session IDs (MUST).** Session IDs MUST be secure and non-deterministic; generated IDs (e.g., UUIDs) SHOULD come from secure random number generators. Avoid predictable or sequential identifiers; rotation and expiry further shrink the window.
- **Bind sessions to the user (SHOULD).** When storing or transmitting session-keyed data (e.g., in a shared queue), combine the session ID with information unique to the authorized user — a key format like `<user_id>:<session_id>`, where the user ID is **derived from the user's token, not supplied by the client**. A guessed session ID then no longer suffices to write into another user's stream.

## Relation to pre-AI IAM

This is the textbook web session-hijacking threat and its textbook controls: high-entropy session identifiers, session expiry/rotation, and never treating session possession as authentication — all standard OWASP session-management guidance, and the same reasoning behind pairing session cookies with re-authorization for sensitive actions. The `<user_id>:<session_id>` binding is the classic "scope the session to its principal server-side" pattern. A practitioner who has hardened a session store already knows every control here.

## Why pre-AI IAM is insufficient

Pre-AI session hijacking yields impersonation: the attacker reads and acts *as* the victim. In an agentic deployment the session also carries **server-to-agent instructions** — asynchronous events, resumed streams, tool-list updates — so a hijacked session inverts into an injection channel: the attacker acts *on* the victim, steering an agent that trusts its transport. The blast radius is no longer bounded by what the attacker can request but by what the victim's agent can be induced to do with its own delegated authority (a [[confused-deputy]] outcome). That is why MCP makes token-verification-per-request and session/authentication separation normative MUSTs rather than hygiene advice, and why queue keys must bind the session to a token-derived user identity.
