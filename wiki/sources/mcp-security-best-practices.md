---
title: MCP Security Best Practices
category: source
status: stable
confidence: high
aliases: [MCP Security Best Practices, security_best_practices, MCP security guide, MCP token passthrough guide]
enterprise_analogs: [RFC 9700 OAuth 2.0 Security BCP, RFC 6819 OAuth threat model, OWASP session management, OWASP SSRF prevention, Content Security Policy, OS consent prompts and sandboxing]
last_updated: 2026-07-08
sources: [raw/MCPSecurityBestPractices.md]
related: [security-considerations, mcp-authorization, confused-deputy, token-passthrough, server-side-request-forgery, session-hijacking, local-mcp-server-security, authorization-url-injection, open-redirection, scope-selection-strategy, step-up-authorization, human-in-the-loop-authorization, rfc-9700-oauth-security-bcp, oauth-2-1, mcp-specification, prompt-injection]
tags: [mcp, security, threat-model, guide, source]
---

# MCP Security Best Practices

Summary of `raw/MCPSecurityBestPractices.md`, pulled 2026-07-08 from <https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices>. Unlike the four documents of the [[mcp-authorization|MCP Authorization specification]], this is a standalone guide in the MCP documentation (a tutorials page, not a spec chapter), but it writes in RFC-2119 normative style and the spec's [[mcp-authorization-security-considerations|Security Considerations]] document defers to it for the fuller treatment of [[token-passthrough]] and the [[confused-deputy]] problem. Its stated audience is developers implementing MCP authorization flows, MCP server operators, and security professionals evaluating MCP-based systems; it is meant to be read alongside the Authorization spec and [[rfc-9700-oauth-security-bcp|RFC 9700, the OAuth 2.0 Security BCP]] (*Introduction*).

## Coverage map

The guide catalogs eight attack classes. Where each lives in the wiki:

| Guide section | Wiki page |
| --- | --- |
| Confused Deputy Problem | [[confused-deputy]] |
| Token Passthrough | [[token-passthrough]] |
| Server-Side Request Forgery | [[server-side-request-forgery]] |
| Session Hijacking | [[session-hijacking]] |
| Local MCP Server Compromise | [[local-mcp-server-security]] |
| OAuth Authorization URL Validation | [[authorization-url-injection]] |
| stdio Transport Security in Proxy Scenarios | [[local-mcp-server-security]] (escalation chain also on [[authorization-url-injection]]) |
| Scope Minimization | [[scope-selection-strategy]] (with [[step-up-authorization]]) |

## Key claims by section

### Confused deputy problem

The guide gives the full anatomy behind the Security Considerations one-liner. An **MCP proxy server** (an MCP server that fronts a third-party API as a single OAuth client, using a **static client ID** at the third-party AS) becomes exploitable when four conditions hold simultaneously: static upstream client ID; downstream MCP clients can **dynamically register**; the third-party AS sets a **consent cookie** after first approval; and the proxy lacks per-client consent (*Vulnerable Conditions*). The attack: a user consents once legitimately; the attacker registers a malicious client with an attacker-controlled `redirect_uri` and sends the user a crafted authorization link; the consent cookie causes the third-party AS to **skip its consent screen**; the proxy mints an MCP authorization code and delivers it to the attacker's redirect URI; the attacker exchanges it for tokens and reaches the third-party API as the user (*Attack Description*).

Mitigations (proxy servers **MUST** implement per-client consent): a per-user registry of approved `client_id` values checked **before** the third-party flow; a consent UI that names the requesting client, displays the third-party scopes, shows the registered `redirect_uri`, and carries CSRF and anti-clickjacking protection; consent cookies with `__Host-` prefix, `Secure`/`HttpOnly`/`SameSite=Lax`, cryptographic signing or server-side sessions, bound to the specific `client_id`; **exact-string** redirect-URI matching (no wildcards); and full OAuth `state` lifecycle — cryptographically random, stored **only after** consent approval and set immediately before the third-party redirect, exactly matched at the callback, single-use, short expiry (~10 minutes). Setting the `state`-bearing cookie before consent approval "renders the consent screen ineffective" (*Mitigation*). See [[confused-deputy]], [[open-redirection]], [[human-in-the-loop-authorization]].

### Token passthrough

Restates the prohibition (an MCP server **MUST NOT** accept tokens not explicitly issued for it) and supplies the risk analysis the spec defers here: **security-control circumvention** (bypassing rate limiting, request validation, monitoring keyed to token audience), **accountability and audit-trail failures** (the server cannot distinguish clients presenting opaque upstream tokens; downstream logs attribute requests to the wrong identity; a stolen token turns the server into a data-exfiltration proxy), **trust-boundary breakage** (a token accepted by multiple services lets compromise of one service pivot into others), and **future-compatibility risk** (a "pure proxy" that later needs controls is stuck without audience separation) (*Risks*). See [[token-passthrough]].

### Server-side request forgery (SSRF)

A **malicious MCP server** can steer an **MCP client** into fetching attacker-chosen URLs during OAuth discovery: the `resource_metadata` URL in `WWW-Authenticate`, the `authorization_servers` in Protected Resource Metadata, and the endpoint URLs in AS metadata. Targets include internal IPs, cloud metadata endpoints (`169.254.169.254`), localhost services, DNS-rebinding domains, and redirect chains (*Attack Description*). Server-deployed MCP clients **MUST** consider SSRF risks; mitigations: **SHOULD** require HTTPS for OAuth URLs ([[oauth-2-1|OAuth 2.1]] §1.5, loopback exempt for development), **SHOULD** block private/reserved IP ranges per [RFC 9728 §7.7](https://datatracker.ietf.org/doc/html/rfc9728#section-7.7) without hand-rolling IP validation, validate redirect targets hop by hop, route through egress proxies (e.g., Smokescreen), and pin DNS between check and use (*Mitigation*). See [[server-side-request-forgery]], which also covers the mirror-image AS-side CIMD fetch surface.

### Session hijacking

Two variants for stateful HTTP deployments: **session hijack prompt injection** (an attacker with a guessed/obtained session ID injects events into a shared queue behind horizontally scaled servers; the payload is resumed/streamed to the legitimate client, which acts on it — including via resumable streams and `notifications/tools/list_changed`) and **session hijack impersonation** (the attacker replays the session ID directly, skipping authentication) (*Attack Description*). Mitigations: authorization-implementing MCP servers **MUST** verify all inbound requests and **MUST NOT** use sessions for authentication; session IDs **MUST** be secure and non-deterministic (**SHOULD** use secure RNGs; rotate/expire); servers **SHOULD** bind session IDs to user-specific information, keying stored session data as `<user_id>:<session_id>` with the user ID derived from the token, not client input (*Mitigation*). See [[session-hijacking]], [[prompt-injection]].

### Local MCP server compromise

Local MCP servers are binaries executed on the user's machine (downloaded, self-authored, or installed via client configuration flows). Attack vectors: malicious "startup" commands embedded in client configuration (one-click installs), malicious payloads inside the server itself, and DNS rebinding against insecure localhost HTTP servers (*Attack Description*). Clients offering one-click configuration **MUST** implement pre-execution consent: show the **exact, untruncated command**, identify it as dangerous, require explicit approval, allow cancellation; **SHOULD** add guardrails (highlight dangerous patterns like `sudo`/`rm -rf`, warn on sensitive-location access, sandbox with minimal default privileges, offer explicit privilege grants, keep sandboxes patched). Servers intended for local use **SHOULD** prevent unauthorized access from other local processes: prefer `stdio` transport; if HTTP, require an authorization token or use unix domain sockets/IPC with restricted access (*Mitigation*). See [[local-mcp-server-security]], [[human-in-the-loop-authorization]].

### OAuth authorization URL validation

Malicious MCP servers can supply authorization URLs that exploit client-side URL handling: `javascript:` URLs passed to `window.open()` yield XSS in the client; URLs opened via shell commands yield command injection and RCE; combined with proxy-managed `stdio` transports, XSS escalates to full system compromise (*Attack Description*). Clients **MUST** allowlist `http://`/`https://` schemes only (`http` for loopback during development only), **MUST** reject `javascript:`, `data:`, `file:`, `vbscript:`; **MUST NOT** open URLs via shells (**SHOULD** use platform-specific, non-shell openers); web-based clients **SHOULD** set CSP (`script-src 'self'`, `default-src 'self'`, nonces where needed); clients **MUST** sanitize and validate all URLs received from servers and log suspicious ones (*Mitigation*). See [[authorization-url-injection]].

### stdio transport security in proxy scenarios

The `stdio` transport is not inherently vulnerable, but in **proxy architectures** — a local proxy service spawns MCP servers as child processes — it becomes an escalation path: client-side XSS (e.g., via a malicious authorization URL) steals the client↔proxy authentication token; the attacker then makes authenticated requests that cause the proxy to spawn arbitrary commands, achieving RCE with user privileges (*Attack Description*). Primary defense is preventing the enabling vulnerability classes ([[authorization-url-injection]], CSP, input sanitization); damage limitation: proxies **SHOULD** sandbox/containerize spawned processes, restrict their filesystem access, log all `stdio` usage, and require additional authorization for dangerous commands; clients **SHOULD** isolate proxy communication, apply least privilege, and sandbox the proxy itself (*Mitigation*). See [[local-mcp-server-security]].

### Scope minimization

Broad up-front scopes (`files:*`, `db:*`, `admin:*`) magnify token-compromise blast radius, complicate revocation, mask per-operation intent in audit logs, enable immediate privilege chaining, and drive consent abandonment (*Risks*). The mitigation is a progressive least-privilege model: a minimal initial scope set (e.g., `mcp:tools-basic`), incremental elevation via targeted `WWW-Authenticate` `scope="..."` challenges, and down-scoping tolerance (servers accept reduced-scope tokens; the AS **MAY** issue a subset of requested scopes). Servers should emit precise challenges (not the full catalog) and log elevation events with correlation IDs; clients should begin with baseline scopes and cache recent denials to avoid elevation loops. Named mistakes: publishing every scope in `scopes_supported`, wildcard/omnibus scopes, bundling unrelated privileges, returning the whole catalog per challenge, silent scope-semantic changes, and treating claimed scopes as sufficient **without server-side authorization logic** (*Common Mistakes*). See [[scope-selection-strategy]], [[step-up-authorization]], [[tool-use-authorization]].

## Relation to pre-AI IAM

The guide is largely classic web and OAuth security applied to MCP topology, and it says so — it points readers at [[rfc-9700-oauth-security-bcp|RFC 9700]] and OWASP references throughout. Confused deputy, consent-cookie CSRF, `state` handling, session-ID entropy, SSRF egress controls, `javascript:`-URI XSS, and shell-injection-safe URL opening are all pre-AI disciplines; a practitioner who has hardened an OAuth proxy, a session store, or a webhook fetcher already owns the mental models. Its contribution is locating each discipline precisely in the MCP architecture: which MCP role is the fetcher, which is the deputy, which holds the session.

## Why pre-AI IAM is insufficient

Three genuinely new pressures recur across the eight sections. First, **the client population is open and machine-driven** — proxies dynamically register downstream clients, clients fetch discovery URLs from servers they have never seen — so protections that classic deployments got implicitly from pre-registration and hard-coded endpoints (consent-per-client, trusted metadata URLs, trusted authorization URLs) must be implemented explicitly. Second, **the client is an agent that acts on what it receives**: a hijacked session is not just impersonation, it is a [[prompt-injection]] delivery channel that can change the agent's tool inventory mid-session. Third, **MCP servers execute on user machines with user privileges**, making software-installation consent and process sandboxing part of the authorization story ([[human-in-the-loop-authorization]]) rather than a separate endpoint-security concern.

## Link

- Guide: <https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices>
