# Wiki Index

## Concepts

- [[agentic-identity]] — domain anchor: who an agent is and whose authority it carries · evolving · medium
- [[authorization-server-mix-up]] — tricking a client into using the wrong AS; defended by RFC 9207 `iss` validation · stub · high
- [[confused-deputy]] — privileged agent tricked into misusing its delegated authority · evolving · high
- [[delegated-authorization]] — an agent acting on a resource on behalf of a consenting user · stable · high
- [[human-in-the-loop-authorization]] — requiring explicit human consent for agent authority/actions · stub · medium
- [[machine-identity]] — an agent's own (workload) identity vs. the user authority it carries · stub · medium
- [[proof-key-for-code-exchange]] — PKCE: binds an auth code to the requesting client; mandatory in MCP · stable · high
- [[step-up-authorization]] — runtime escalation of scopes via `insufficient_scope` challenges · stable · high
- [[token-audience-binding]] — issuing/validating tokens for a specific resource audience (RFC 8707) · stable · high
- [[token-passthrough]] — anti-pattern: accepting or forwarding tokens not issued for you · stub · high
- [[tool-use-authorization]] — what an agent may do when it invokes a tool; scopes as the unit · stub · medium

## Topics

- [[authorization-server-discovery]] — finding/validating the AS via RFC 9728 → RFC 8414 / OIDC · evolving · high
- [[canonical-server-uri]] — the RFC 8707 `resource` parameter and canonical URI rules for MCP servers · stable · high
- [[client-registration]] — Client ID Metadata Documents, pre-registration, deprecated DCR · evolving · high
- [[mcp-authorization]] — MCP's transport-level authorization model, a profile of OAuth 2.1 · evolving · high
- [[scope-selection-strategy]] — least-privilege scope selection for domain-blind clients · stable · high
- [[security-considerations]] — normative threat mitigations (forward stub; doc 4 not yet ingested) · stub · high

## References

- [[mcp-authorization-extensions]] — optional, additive, composable auth extensions (ext-auth repo) · stub · high
- [[mcp-specification]] — the Model Context Protocol open spec; authorization is one optional chapter · evolving · high
- [[oauth-2-1]] — consolidated OAuth 2.0 + best practices; MCP's baseline · evolving · high
- [[oauth-client-id-metadata-documents]] — URL-as-`client_id`; preferred MCP client identity (draft) · evolving · high
- [[openid-connect-discovery]] — `.well-known/openid-configuration` AS discovery · stable · high
- [[openid-connect-dynamic-client-registration]] — OIDC profile of runtime client registration · stable · high
- [[rfc-6750-bearer-token-usage]] — bearer header, `WWW-Authenticate`, `insufficient_scope` · stable · high
- [[rfc-7591-dynamic-client-registration]] — runtime `/register` client registration (deprecated in MCP) · stable · high
- [[rfc-7636-pkce]] — Proof Key for Code Exchange spec · stable · high
- [[rfc-8414-authorization-server-metadata]] — `.well-known/oauth-authorization-server` AS discovery · stable · high
- [[rfc-8707-resource-indicators]] — `resource` parameter for audience-restricted tokens · stable · high
- [[rfc-9207-authorization-server-issuer-identification]] — `iss` parameter; mix-up defense · stable · high
- [[rfc-9728-protected-resource-metadata]] — `.well-known/oauth-protected-resource`; mandatory on MCP servers · stable · high

## Entities

- [[anthropic]] — AI company that originated the Model Context Protocol · stub · medium
- [[ietf-oauth-working-group]] — IETF body producing the OAuth RFCs/drafts MCP depends on · stub · high
- [[openid-foundation]] — standards body for OpenID Connect (Discovery, Dynamic Registration) · stub · high

## Incidents

_None yet._

## Sources

- [[mcp-authorization-overview]] — summary of MCP Authorization spec, doc 1 of 4 (Overview) · stable · high
