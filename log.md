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

## [2026-06-19] ingest | MCP Authorization Specification — Authorization Server Discovery (doc 2 of 4)
Ingested raw/MCPAuthorization_AuthorizationServerDiscovery.md; 1 new source page, 7 pages updated.
- mcp-authorization-server-discovery: new source page summarizing doc 2 (AS location, two PRM discovery mechanisms, AS-metadata endpoint construction matrix, metadata-issuer validation)
- authorization-server-discovery: major rewrite — removed "not yet ingested" stub note; added Stage 1 (RFC 9728 location, multi-AS selection per §7.6, per-AS state per RFC 6749 §2.2) and Stage 2 (path/no-path endpoint priority tables, default suffix); documented the two distinct issuer validations; status evolving → stable
- authorization-server-mix-up: added Defense 1 (metadata-document issuer validation, RFC 8414 §3.3 / OIDC §4.3) alongside existing Defense 2 (RFC 9207 response iss); added rfc-8414 to related; remains stub pending doc 4
- rfc-9728-protected-resource-metadata: added authorization_servers field, two location mechanisms (WWW-Authenticate vs well-known + fallback order), multi-AS selection (§7.6); added client-registration to related
- rfc-8414-authorization-server-metadata: added default-suffix + path-insertion construction (§3.1/§5) and §3.3 issuer validation; added authorization-server-mix-up to related
- openid-connect-discovery: added path-insertion vs path-appending endpoint shapes and §4.3 validation; added authorization-server-mix-up to related
- client-registration: added "Authorization Server Binding (per-AS state)" section forward-referencing doc 3; added doc-2 source
- mcp-authorization: added mcp-authorization-server-discovery to sources (discovery claims now backed by doc 2)
- _index.md: added source line for doc 2; bumped authorization-server-discovery to stable
