---
title: Client Registration
category: topic
status: evolving
confidence: high
aliases: [MCP client registration, client ID metadata documents, dynamic client registration, DCR]
enterprise_analogs: [RFC 7591 Dynamic Client Registration, OIDC Dynamic Client Registration 1.0, manual client pre-registration, OAuth Client ID Metadata Documents draft]
last_updated: 2026-06-18
sources: [mcp-authorization-overview]
related: [mcp-authorization, oauth-client-id-metadata-documents, rfc-7591-dynamic-client-registration, openid-connect-dynamic-client-registration, authorization-server-discovery]
tags: [oauth, client-registration, dcr, mcp]
---

# Client Registration

**Client registration** is how an [[mcp-authorization|MCP client]] obtains a `client_id` to use with an authorization server. Because MCP clients connect to servers with which they have no prior relationship, the spec defines three mechanisms with a selection priority ([[mcp-authorization-overview]], *Client Registration*).

> **Status: evolving.** This page summarizes the [[mcp-authorization-overview|overview]]. The dedicated specification document (`raw/MCPAuthorization_ClientRegistration.md`) is present but **not yet ingested**; the full selection priority and normative detail will be added then.

## The three mechanisms

1. **[[oauth-client-id-metadata-documents|Client ID Metadata Documents]]** (draft-ietf-oauth-client-id-metadata-document-00) — **SHOULD** be supported by both AS and clients. The client uses an **HTTPS URL as its `client_id`**; the AS detects the URL-formatted ID, fetches the metadata document from it, and validates the metadata and `redirect_uris`. No prior registration call is needed.
2. **Pre-registration** — the client uses an existing, out-of-band-issued `client_id`.
3. **[[rfc-7591-dynamic-client-registration|Dynamic Client Registration (RFC 7591)]]** — **MAY** be supported; the client POSTs to `/register` and receives client credentials. **Deprecated** in this profile, "retained for backwards compatibility with authorization servers that do not support Client ID Metadata Documents." The OIDC variant is [[openid-connect-dynamic-client-registration]].

Before initiating the authorization flow, clients MUST obtain a `client_id` through one of these three, following the selection priority defined in the dedicated document.

## Relation to pre-AI IAM

Pre-registration is the classic enterprise model (an admin registers each app and gets a client ID/secret). RFC 7591 / OIDC Dynamic Client Registration automated that for federations. Client ID Metadata Documents are the newest turn: a publicly dereferenceable URL *is* the identity, akin in spirit to how SAML entities are identified by a metadata URL.

## Why pre-AI IAM is insufficient

Manual pre-registration assumes a **bounded, known set of clients** an administrator can enroll ahead of time. An MCP ecosystem has an **open, unbounded client population** meeting **open, unbounded servers** — no admin can pre-register every agent against every server. The profile's answer is registration-free identity: Client ID Metadata Documents let a client present a URL the AS can resolve on the spot, and it explicitly **deprecates** classic Dynamic Client Registration (whose unauthenticated `/register` endpoint is itself an abuse surface) in favor of it. This is the registration analog of the same shift seen in [[authorization-server-discovery]]: replace human-time integration with runtime, self-describing discovery.
