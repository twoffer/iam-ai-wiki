---
title: Authorization URL Injection
category: concept
status: stable
confidence: high
aliases: [OAuth authorization URL validation, javascript URL injection, malicious authorization URL, XSS via authorization URL, command injection via URL opening]
enterprise_analogs: [DOM-based XSS via javascript: URIs, OS command injection (CWE-78), URL scheme allowlisting, Content Security Policy, RFC 8252 §8 native-app browser hand-off]
last_updated: 2026-07-08
sources: [mcp-security-best-practices]
related: [mcp-authorization, open-redirection, local-mcp-server-security, session-hijacking, security-considerations, token-theft]
tags: [security, oauth, xss, rce, threat-model, core-concept]
---

# Authorization URL Injection

**Authorization URL injection** is the attack in which a malicious MCP server supplies an OAuth authorization URL crafted to exploit how the MCP client *opens* it. During the [[mcp-authorization|authorization flow]] the client receives the authorization endpoint URL from server-controlled metadata and hands it to a browser or OS facility; if the client does not validate the URL, the hand-off itself becomes the vulnerability ([[mcp-security-best-practices]], *OAuth Authorization URL Validation*).

## Attack vectors

- **JavaScript URL injection (XSS).** The server presents a `javascript:` URL as the authorization endpoint; the client passes it to `window.open()` or a similar browser API; the embedded script executes in the client application's context — enabling session hijacking, credential theft, and further exploitation.
- **Command injection via shell execution.** The server embeds shell metacharacters in the URL; a client that opens URLs through `cmd.exe`, PowerShell, or a shell script lets the shell interpret parts of the URL as commands — arbitrary code execution with the user's privileges.
- **Escalation through stdio proxies.** Where the client talks to a local proxy that spawns MCP servers as `stdio` child processes, XSS in the client can steal the client↔proxy authentication token and direct the proxy to spawn arbitrary commands — full system compromise from a web-context foothold. The chain is detailed on [[local-mcp-server-security]].

The resulting risks span XSS, RCE, privilege escalation, data exfiltration, and persistence ([[mcp-security-best-practices]], *Risks*).

## Mitigations

- **Scheme allowlisting (MUST).** Clients MUST accept only `http://` and `https://` authorization URLs (`http` solely for loopback addresses during development; production authorization servers MUST use `https`), MUST reject `javascript:`, `data:`, `file:`, `vbscript:` and other dangerous schemes, and SHOULD use allowlist- rather than blocklist-based validation.
- **No shell in the open path (MUST NOT).** Clients MUST NOT use shell commands to open URLs and SHOULD use platform-specific, non-shell URL-opening mechanisms.
- **Content Security Policy (SHOULD).** Web-based clients SHOULD set CSP headers — `script-src 'self'`, `default-src 'self'`, nonces for necessary inline scripts — so an injected `javascript:` payload has nowhere to run.
- **Input sanitization (MUST).** Clients MUST sanitize and validate every URL received from an MCP server: strict parsing, rejection of shell-significant characters, dedicated sanitization libraries, and logging of suspicious authorization URLs for monitoring.

## Relation to pre-AI IAM

The ingredients are classic application security rather than OAuth per se: `javascript:`-URI XSS is textbook DOM-based XSS, shell-mediated URL opening is command injection (CWE-78), and the mitigations — scheme allowlists, CSP, never composing shell strings from untrusted input — are standard secure-coding controls. The OAuth-native cousin is the native-app guidance (RFC 8252) that clients hand authorization URLs to the system browser; that guidance simply assumed the URL pointed at a legitimate AS. [[open-redirection]] is the adjacent OAuth threat, but it abuses *where the AS redirects afterward*; this attack abuses *the URL the client opens first*.

## Why pre-AI IAM is insufficient

In classic OAuth the authorization URL is effectively **configuration**: the developer registers with a known AS and ships its endpoint, so "validate the URL you are about to open" was never a meaningful control. MCP inverts the trust direction — the client learns the authorization endpoint at runtime, from [[mcp-authorization|discovery metadata controlled by the very server it just met]], which may be adversarial. Every URL in the flow is therefore untrusted input on par with request parameters, and the client must apply input-validation discipline to what used to be a constant. The same inversion drives the client-side [[server-side-request-forgery|SSRF]] surface (which URLs the client *fetches*); this page covers which URLs the client *opens*.
