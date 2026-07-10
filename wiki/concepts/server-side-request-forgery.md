---
title: Server-Side Request Forgery (SSRF)
category: concept
status: stable
confidence: high
aliases: ["SSRF", "server-side request forgery", "AS abuse protection", "metadata fetch SSRF", "discovery metadata SSRF"]
enterprise_analogs: ["classic web SSRF (OWASP A10:2021)", "OAuth Client ID Metadata Document draft §6 SSRF", "RFC 9728 §7.7 private-range blocking", "outbound-fetch allowlisting", "egress proxies", "cloud metadata endpoint hardening"]
last_updated: 2026-07-08
sources: ["mcp-security-best-practices", "mcp-authorization-security-considerations"]
related: ["oauth-client-id-metadata-documents", "client-registration", "security-considerations", "mcp-authorization", "machine-identity", "authorization-server-discovery", "rfc-9728-protected-resource-metadata", "oauth-2-1", "authorization-url-injection"]
tags: ["security", "ssrf", "cimd", "discovery", "threat-model", "core-concept"]
---

# Server-Side Request Forgery (SSRF)

**Server-Side Request Forgery (SSRF)** is the threat that an attacker induces a machine to make outbound requests to URLs of the attacker's choosing — typically to reach internal services the attacker cannot address directly (private administration endpoints, cloud metadata services, other hosts inside the trust boundary). MCP's registration-free, discovery-driven authorization model creates **two** SSRF surfaces, one on each side of the protocol: the **authorization server** dereferencing attacker-supplied [[oauth-client-id-metadata-documents|Client ID Metadata Document (CIMD)]] URLs ([[mcp-authorization-security-considerations]], *Authorization Server Abuse Protection*), and the **MCP client** dereferencing attacker-supplied OAuth discovery URLs ([[mcp-security-best-practices]], *Server-Side Request Forgery*).

## Surface 1: the AS fetching CIMD URLs

CIMD lets a client present an HTTPS URL as its `client_id`; the AS dereferences that URL to fetch the client's metadata document. Because the URL is supplied by an **unknown, unauthenticated client**, a malicious client can point the AS at an arbitrary URL — for example a private admin endpoint the AS can reach — and use the AS as a confused fetcher.

The mitigation is advisory in the draft: ASes that fetch metadata documents **SHOULD** consider SSRF risks ([draft §6, "Server-Side Request Forgery"](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00#name-server-side-request-forgery)). Practical controls are the standard SSRF defenses applied to the metadata fetch: restrict outbound fetches to public HTTPS endpoints; block private/link-local/loopback ranges and cloud metadata IPs; apply [[client-registration|trust policies]] (domain allowlists, reputation checks, certificate/domain-age restrictions) before fetching an unknown `client_id` URL; and bound redirects, response size, and timeouts. This composes with the other CIMD-security rules in [[security-considerations]] (`localhost` redirect-URI warnings, exact `redirect_uri` matching, prominent hostname display).

## Surface 2: the MCP client fetching discovery metadata

The mirror image, from the [[mcp-security-best-practices|Security Best Practices]] guide: during [[authorization-server-discovery|OAuth metadata discovery]] the MCP client fetches URLs that a **malicious MCP server** controls — the `resource_metadata` URL in the `WWW-Authenticate` header, the `authorization_servers` URLs in [[rfc-9728-protected-resource-metadata|Protected Resource Metadata]], and the `token_endpoint`/`authorization_endpoint` URLs in AS metadata. Attack patterns: direct internal-IP targets (`http://192.168.1.1/admin`), **cloud metadata endpoints** (`http://169.254.169.254/` — exfiltrating IAM credentials and instance data), localhost services (`http://localhost:6379/` reaching Redis or admin panels), **DNS rebinding** (a domain resolving safely at validation time and internally at request time), and redirect chains landing on internal resources.

The risks go beyond reads: cloud-credential exfiltration, internal-network reconnaissance via error messages, state-changing **POSTs** (a token-endpoint call is a POST the attacker aims at an internal service), perimeter/firewall bypass with the client as proxy, and data reflection back to the attacker through error details or OAuth flow responses.

Server-deployed MCP clients **MUST** consider SSRF risks and mitigate appropriately for their network environment ([[mcp-security-best-practices]], *Mitigation*):

- **Enforce HTTPS (SHOULD).** Reject `http://` OAuth URLs except loopback during development — aligned with [[oauth-2-1|OAuth 2.1]] §1.5 — with an explicit opt-out for dev/testing.
- **Block private ranges (SHOULD).** Per [RFC 9728 §7.7](https://datatracker.ietf.org/doc/html/rfc9728#section-7.7): private IPv4 (`10/8`, `172.16/12`, `192.168/16`), loopback, link-local `169.254/16` (cloud metadata), and private IPv6 (`fc00::/7`, `fe80::/10`). Do **not** hand-roll IP validation — encoding tricks (octal, hex, IPv4-mapped IPv6) defeat custom parsers.
- **Validate redirect targets (SHOULD).** Apply the same scheme/range checks to every redirect hop; consider disabling automatic redirect following.
- **Use egress proxies.** Route discovery fetches through a proxy that blocks internal destinations by design (e.g., [Smokescreen](https://github.com/stripe/smokescreen)); restrict the client's outbound network policy.
- **Pin DNS between check and use.** DNS-based validation is TOCTOU-prone (rebinding); pin the resolved address for the actual request and treat DNS checks as one layer of defense in depth.

## Relation to pre-AI IAM

SSRF is a classic web-application vulnerability (OWASP A10:2021; see the [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)), not novel to OAuth. The mitigations — egress allowlisting, blocking internal address ranges, redirect and DNS-rebinding discipline — are exactly what a practitioner applies to any feature that fetches a user-supplied URL (webhooks, link unfurlers, document importers). MCP imports that discipline into the OAuth control plane on both sides of the protocol.

## Why pre-AI IAM is insufficient

Classic OAuth gave neither party an attacker-controllable fetch. Client identities were pre-registered out of band, so the AS only ever talked to its own configured endpoints; and the client's AS endpoints were compiled-in configuration, so "where do I fetch metadata from?" was never a runtime question. MCP's open-population model changes both: **registration-free client identity** makes the AS dereference URLs from unauthenticated strangers, and **runtime AS discovery** makes the client dereference URLs from servers it met moments ago — often from privileged network positions (CI runners, cloud instances with metadata services) an agent deployment tends to occupy. The controls aren't new; what is new is that both the AS and the agent-side client must now carry a hardened URL-fetching pipeline as core OAuth machinery. The companion threat — URLs the client *opens in a browser* rather than fetches — is [[authorization-url-injection]].
