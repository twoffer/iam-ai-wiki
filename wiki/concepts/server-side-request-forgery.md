---
title: Server-Side Request Forgery (SSRF)
category: concept
status: stable
confidence: high
aliases: [SSRF, server-side request forgery, AS abuse protection, metadata fetch SSRF]
enterprise_analogs: [classic web SSRF, OAuth Client ID Metadata Document draft §6 SSRF, outbound-fetch allowlisting, metadata-endpoint hardening]
last_updated: 2026-06-19
sources: [mcp-authorization-security-considerations]
related: [oauth-client-id-metadata-documents, client-registration, security-considerations, mcp-authorization, machine-identity]
tags: [security, ssrf, cimd, threat-model, core-concept]
---

# Server-Side Request Forgery (SSRF)

**Server-Side Request Forgery (SSRF)** is the threat that an attacker induces a server to make outbound requests to URLs of the attacker's choosing — typically to reach internal services the attacker cannot address directly (private administration endpoints, cloud metadata services, other hosts inside the trust boundary). In the MCP Authorization profile, SSRF appears as a new attack surface created by [[oauth-client-id-metadata-documents|Client ID Metadata Documents (CIMD)]]: the authorization server "takes a URL as input from an unknown client and fetches that URL" ([[mcp-authorization-security-considerations]], *Authorization Server Abuse Protection*).

## How it arises in MCP

CIMD lets a client present an HTTPS URL as its `client_id`; the AS dereferences that URL to fetch the client's metadata document. Because the URL is supplied by an **unknown, unauthenticated client**, a malicious client can point the AS at an arbitrary URL — for example a private admin endpoint the AS can reach — and use the AS as a confused fetcher.

## How MCP mitigates it

The mitigation is advisory in the draft: ASes that fetch metadata documents **SHOULD** consider SSRF risks ([draft §6, "Server-Side Request Forgery"](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-client-id-metadata-document-00#name-server-side-request-forgery)). Practical controls are the standard SSRF defenses applied to the metadata fetch:

- Restrict outbound fetches to public HTTPS endpoints; block private/link-local/loopback ranges and cloud metadata IPs.
- Apply [[client-registration|trust policies]] — domain allowlists, reputation checks, certificate/domain-age restrictions — before fetching an unknown `client_id` URL.
- Bound redirects, response size, and timeouts on the fetch.

This composes with the other CIMD-security rules in [[security-considerations]] (`localhost` redirect-URI warnings, exact `redirect_uri` matching, prominent hostname display).

## Relation to pre-AI IAM

SSRF is a classic web-application vulnerability (OWASP), not novel to OAuth. The mitigations — egress allowlisting, blocking internal address ranges, and validating fetch targets — are exactly what a practitioner applies to any feature that fetches a user-supplied URL (webhooks, link unfurlers, document importers). The CIMD draft simply imports that discipline into the OAuth control plane.

## Why pre-AI IAM is insufficient

Classic OAuth never had the authorization server **fetch a URL chosen by an unauthenticated client** — client identities were pre-registered out of band, so the AS only ever talked to its own configured endpoints. Registration-free client identity changes that: to support an open, runtime-discovered client population without a human integrator, the AS must dereference attacker-controllable URLs on demand, importing SSRF into a component that previously had no outbound-fetch attack surface. The control isn't new; the requirement to apply it *inside the AS* is what the agentic, registration-free model introduces. See [[oauth-client-id-metadata-documents]] and [[client-registration]].
