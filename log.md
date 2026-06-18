# Log

## [2026-06-18] ingest | MCP Authorization Specification — Overview (doc 1 of 4)
First source ingested; used to seed the wiki from empty. 34 pages created.
- mcp-authorization-overview: new source page summarizing raw/MCPAuthorization_Overview.md (the OAuth 2.1 profile, roles, flow, normative claims)
- mcp-authorization: new central topic — MCP's transport-level OAuth 2.1 authorization profile, with full pre-AI IAM bridging
- agentic-identity: new concept (domain anchor) — agent identity vs. delegated user authority
- delegated-authorization: new concept — agent acting on behalf of a consenting user
- token-audience-binding: new concept — mandatory RFC 8707 audience restriction + server validation
- confused-deputy: new concept — privileged agent tricked into misusing delegated authority
- proof-key-for-code-exchange: new concept — PKCE, mandatory via OAuth 2.1
- step-up-authorization: new concept — runtime scope escalation via insufficient_scope
- machine-identity: new stub concept — workload identity (STDIO env creds, client_credentials)
- human-in-the-loop-authorization: new stub concept — user consent step
- tool-use-authorization: new stub concept — scopes as the unit of tool-call authority
- token-passthrough: new stub concept — anti-pattern; MUST NOT accept/transit foreign tokens
- authorization-server-mix-up: new stub concept — mix-up attack; RFC 9207 iss defense
- authorization-server-discovery: new evolving topic — RFC 9728 → RFC 8414/OIDC (doc 2 pending)
- client-registration: new evolving topic — CIMD / pre-reg / deprecated DCR (doc 3 pending)
- scope-selection-strategy: new topic — least-privilege scope selection for domain-blind clients
- canonical-server-uri: new topic — resource parameter + canonical URI rules (RFC 8707)
- security-considerations: new stub topic — forward stub for doc 4 (Security Considerations) pending
- references: added oauth-2-1, rfc-6750, rfc-7591, rfc-7636, rfc-8414, rfc-8707, rfc-9207, rfc-9728, oauth-client-id-metadata-documents, openid-connect-discovery, openid-connect-dynamic-client-registration, mcp-specification, mcp-authorization-extensions (13)
- entities: added anthropic, ietf-oauth-working-group, openid-foundation (3)
- _index.md: rebuilt with all 34 pages
- note: no incident pages — the overview describes threat classes (confused-deputy, mix-up), not real-world incidents
