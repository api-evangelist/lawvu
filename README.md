# LawVu

LawVu is a New Zealand-founded legal technology company whose LegalOS is a connected workspace for
corporate in-house legal teams — matter management, contract lifecycle management, legal request
intake, spend management and e-billing, document management, reporting and embedded AI in one
platform.

LawVu publishes a public REST developer platform: a **v2 API** covering matters, contracts, fields
and files, plus a set of **legacy v1 product APIs** covering accounts, users, organisations, email
threads, invoices and webhook subscriptions. It is secured with OAuth 2.0 authorization code,
documented with OpenAPI 3.0.1, returns RFC 9457 problem details, and filters, sorts and pages with
a subset of OData. LawVu also ships a first-party MCP server for connecting AI assistants to the
LegalOS, a Microsoft Power Platform connector, and a Workato connector.

- Website — https://lawvu.com/
- Developer portal — https://developer.lawvu.com/
- Documentation & API reference — https://api-docs.lawvu.com/docs
- Trust Center — https://lawvu.com/trust-center/
- System status — https://lawvu.com/lawvu-system-status/
- Developer contact — developer@lawvu.com

## Artifacts in this repo

| Area | Artifact |
|---|---|
| Specifications | `openapi/` — 7 OpenAPI 3.0.1 documents (v2 API + 6 legacy product APIs) |
| Enhancements | `overlays/` — one OpenAPI Overlay 1.0.0 per spec |
| Authentication | `authentication/` — OAuth 2.0 authorization code profile |
| Conventions | `conventions/` — OData paging/filtering, error envelope, versioning, tracing |
| Errors | `errors/` — RFC 9457 problem types, OData and OAuth error references |
| Events | `asyncapi/` — the 16-topic webhook catalog (no AsyncAPI is published) |
| Lifecycle | `lifecycle/` — versioning, v1→v2 posture, 99.95% SLA, status page |
| Changelog | `changelog/` — dated entries from the published developer changelog |
| Sandbox | `sandbox/` — sandbox vs production environments and promotion path |
| Agents | `mcp/`, `skills/`, `llms/`, `agentic-access/` |
| Security | `security/`, `well-known/`, `conformance/` |
| Packages | `packages/` — connectors; LawVu ships no first-party API client SDK |
| Data model | `data-model/` — entity graph derived from the v2 specification |

Backed by: insight-partners
