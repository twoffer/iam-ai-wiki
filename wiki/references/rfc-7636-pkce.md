---
title: RFC 7636 — Proof Key for Code Exchange (PKCE)
category: reference
status: stable
confidence: high
aliases: ["RFC 7636", "PKCE spec", "code_challenge", "code_verifier"]
enterprise_analogs: []
last_updated: 2026-06-19
sources: ["mcp-authorization-security-considerations", "mcp-authorization-overview"]
related: ["proof-key-for-code-exchange", "oauth-2-1", "mcp-authorization", "rfc-8414-authorization-server-metadata", "security-considerations"]
tags: ["oauth", "pkce", "security", "spec", "ietf", "reference"]
---

# RFC 7636 — Proof Key for Code Exchange (PKCE)

**RFC 7636** defines PKCE: the client sends a `code_challenge` (the hash of a secret `code_verifier`) in the authorization request and proves possession of the verifier at the token endpoint, binding the authorization code to the requesting client and defeating code interception/injection.

## Role in MCP Authorization

PKCE is **mandatory** in MCP because [[oauth-2-1|OAuth 2.1]] (which the profile requires) folds RFC 7636 in and makes it required for all authorization-code clients. The [[mcp-authorization-overview]] flow includes "Generate PKCE parameters" as a required step. The [[mcp-authorization-security-considerations|Security Considerations]] document adds two specifics: clients MUST use the **`S256`** challenge method when capable, and MUST **verify PKCE support** via AS metadata (`code_challenge_methods_supported` in [[rfc-8414-authorization-server-metadata|RFC 8414]] / OIDC metadata) and refuse to proceed if it is absent. See the concept page [[proof-key-for-code-exchange]] for how it fits the flow and composes with `iss` validation.

## Link

- RFC: <https://datatracker.ietf.org/doc/html/rfc7636>
