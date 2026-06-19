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

## [2026-06-19] ingest | MCP Authorization Specification — Client Registration (doc 3 of 4)
Ingested raw/MCPAuthorization_ClientRegistration.md; 1 new source page, 7 pages updated.
- mcp-authorization-client-registration: new source page summarizing doc 3 (three mechanisms + selection priority; CIMD client/AS requirements, example doc, advertising; pre-registration; deprecated DCR + application_type constraints; Authorization Server Binding)
- client-registration: major rewrite — removed "not yet ingested" stub note; added mechanism table, four-step selection priority, full CIMD client/AS requirements, pre-registration forms, DCR application_type rules, and the now-filled Authorization Server Binding section; status evolving → stable
- oauth-client-id-metadata-documents: added "Requirements under the MCP profile" (https+path client_id, required fields, client_id-matches-URL, private_key_jwt §6.2, AS fetch/validate/cache) and CIMD portability across ASes; added doc-3 source and rfc-8414 to related
- rfc-7591-dynamic-client-registration: added "Constraints under the MCP profile" (application_type native/web, registration_endpoint gating, AS-binding re-registration); added doc-3 source and rfc-8414 to related
- openid-connect-dynamic-client-registration: added "The application_type parameter" section (source of native/web semantics; OIDC default of web breaks localhost redirects); added doc-3 source
- rfc-8414-authorization-server-metadata: added "Capability signals for client registration" (client_id_metadata_document_supported and registration_endpoint as selection-priority signals); added doc-3 source and client-registration/oauth-client-id-metadata-documents to related
- mcp-authorization: added mcp-authorization-client-registration to sources (client-registration claims now backed by doc 3)
- mcp-authorization-overview: updated "Notable for future ingests" — doc 3 now ingested; only security-considerations (doc 4) remains a forward stub
- _index.md: added source line for doc 3; bumped client-registration to stable

## [2026-06-19] ingest | MCP Authorization Specification — Security Considerations (doc 4 of 4)
Ingested raw/MCPAuthorization_SecurityConsiderations.md (final doc); 1 new source page, 3 new concepts, 2 new references, 14 pages updated. Completes the MCP Authorization spec in the wiki.
- mcp-authorization-security-considerations: new source page summarizing doc 4 (the normative OAuth threat-model checklist: nine threat areas, full bridging)
- security-considerations: major rewrite — removed "not yet ingested" stub note; added all nine threat sections (audience binding, token theft, communication security, PKCE + PKCE-support discovery, mix-up, open redirection, CIMD security, confused deputy, access-token privilege restriction) with normative detail and full bridging; status stub → stable
- token-theft: new concept — secure storage (OAuth 2.1 §7.1), short-lived tokens, refresh-token rotation for public clients (§4.3.1)
- open-redirection: new concept — registered + exact-match redirect URIs, `state`, untrusted-URI precautions (OAuth 2.1 §7.12.2)
- server-side-request-forgery: new concept — SSRF surface created by AS fetching attacker-supplied CIMD URLs (draft §6); registration-free identity as the new wrinkle
- rfc-9068-jwt-access-tokens: new reference — JWT access-token profile; the `aud` claim underpinning server-side audience validation
- mcp-security-best-practices: new stub reference — the non-spec MCP guide doc 4 repeatedly defers to (token passthrough, confused deputy); not ingested
- token-passthrough: stub → stable — added two-failure-mode decomposition and the upstream-API "two hats" rule (MUST NOT pass through; mint a separate token as OAuth client to upstream)
- confused-deputy: evolving → stable — added the proxy-server / static-client-ID / consent-per-dynamically-registered-client rule; removed forward-stub note
- authorization-server-mix-up: stub → stable — added "why weaker measures don't suffice" (PKCE alone fails, resource indicators don't help, depends on honest AS emitting `iss`); removed forward-stub note
- proof-key-for-code-exchange: added mandatory PKCE-support discovery (`code_challenge_methods_supported`, refuse-to-proceed downgrade defense) and `S256` requirement
- token-audience-binding: added the Access Token Privilege Restriction two-dimensions framing (audience-validation failure vs passthrough); RFC 9728 §7.4
- rfc-8414-authorization-server-metadata: added `code_challenge_methods_supported` as PKCE-support signal
- openid-connect-discovery: added the MCP requirement to verify/include `code_challenge_methods_supported` (non-standard-but-mandatory field)
- oauth-client-id-metadata-documents: added draft §6 security (SSRF, localhost impersonation, trust policies)
- oauth-2-1: added "Security sections MCP requires" table (§1.5/§4.1.1/§4.3.1/§5.2/§7.1/§7.5/§7.12.2)
- rfc-9728-protected-resource-metadata: added §7.4 alignment for the mandatory `resource` parameter
- rfc-7636-pkce / rfc-8707-resource-indicators / rfc-9207-authorization-server-issuer-identification: added doc-4 detail and source cross-refs
- mcp-authorization / client-registration / canonical-server-uri: added doc-4 source + security cross-refs; mcp-authorization sub-pages note de-stubbed
- mcp-authorization-overview: "Notable for future ingests" — all four spec docs now ingested; flagged Security Best Practices guide as the remaining un-ingested external doc
- sibling source pages (server-discovery, client-registration): added doc-4 source to related for graph connectivity
- _index.md: added doc-4 source + 3 concepts + 2 references; bumped authorization-server-mix-up, confused-deputy, token-passthrough, security-considerations to stable
- note: no incident pages — doc 4 describes threat classes and mitigations, not real-world incidents

## [2026-06-19] query | Public vs. confidential MCP OAuth clients & the PKCE flow
- public-vs-confidential-client: new concept — answers whether MCP assumes always-public (no: spec mandates AS support for "both confidential and public clients", `raw/MCPAuthorization_Overview.md:68`, with a `private_key_jwt` confidential path via CIMD), what public means in practice (`token_endpoint_auth_method: none`, no client_credentials, mandatory PKCE/rotation/exact-redirect), and how PKCE differs by client type (PKCE as sole code-binding defense for public vs defense-in-depth for confidential)
- proof-key-for-code-exchange: added cross-ref + note that PKCE is primary (not defense-in-depth) for public clients
- oauth-2-1: added cross-ref on the "both confidential and public clients" baseline clause
- mcp-authorization: added cross-ref + "MCP clients are typically public" note on the OAuth 2.1 building block
- client-registration / oauth-client-id-metadata-documents / token-theft / machine-identity: added reciprocal `related` cross-refs
- _index.md: added public-vs-confidential-client under Concepts
