---
name: Create and track a LawVu matter
description: Open a new legal matter in LawVu, attach the intake document, and keep it in sync with an external system of record.
api: openapi/lawvu-api-openapi-original.yml
operations:
  - get-v2-mattertypes
  - post-v2-matters
  - post-v2-files
  - get-v2-matters-matterid
  - patch-v2-matters-matterid
  - get-v2-matters
---

# Create and track a LawVu matter

A matter is the central unit of legal work in LawVu. Use this skill when a request from the
business needs to become a tracked legal matter.

## Before you start

- Authenticate with OAuth 2.0 authorization code. Every call carries
  `Authorization: Bearer {access_token}`. See `authentication/lawvu-authentication.yml`.
- All calls run under the security context of the user who granted consent — that user's
  permissions decide what you can read and write. If you get a `403`, the user lacks the
  permission, not the token.
- Build against the sandbox first: `https://api-sandbox.lawvu.com` with the web host
  `demo.lawvu.com`. Production is `https://api.lawvu.com` / `go.lawvu.com`.

## Steps

1. **Pick the matter type.** Call `get-v2-mattertypes` and choose the type by name. Types are
   customer-defined and hierarchical, so check `parent` when several share a label. Do not
   hard-code type ids across tenants.

2. **Create the matter.** Call `post-v2-matters` with at minimum `type` and `name`. Set
   `externalId` to the identifier of the originating record in your system — this is what makes
   the matter re-findable later without storing LawVu ids. Set `owner`, `teamAssigned` and
   custom `fields` when you have them. The response is a `CreatedMatter` carrying only `id`.

3. **Attach the intake document.** Call `post-v2-files` with the file, `targetResourceType` set
   to the matter resource type and `targetResourceId` set to the new matter id. Send it as
   multipart form data — a wrong `Content-Type` returns `415`.

4. **Read it back.** Call `get-v2-matters-matterid` to confirm the created state and capture the
   `displayId` your users will recognise.

5. **Keep it in sync.** On later changes call `patch-v2-matters-matterid` with only the fields
   that changed (`name`, `owner`, `manager`, `status`, `teamAssigned`, `fields`, `restricted`,
   `externalId`).

6. **Find it again.** Call `get-v2-matters` with
   `$filter=ExternalId eq '{your-id}'` rather than storing the LawVu id, if your system is the
   source of truth.

## Conventions that apply

- **Pagination and filtering** use an OData subset: `$top` (default 30, max 200), `$skip`,
  `$filter`, `$orderby`. Page with `$top` + `$skip`; never assume the whole collection came back.
- **Filtering** supports `eq`, `ne`, `gt`, `ge`, `lt`, `le`, `contains`, null comparisons, `and`,
  `or`, parentheses, and `/` for nested paths such as `Owner/Id`. Not every field supports every
  operator — an unsupported one returns `400` with a `detail` naming the field or operator.
- **There is no idempotency key.** Re-issuing `post-v2-matters` creates a second matter. Before
  retrying a create whose response you did not see, search by `externalId` first.
- **Errors are RFC 9457** `application/problem+json` with `type`, `title`, `status`, `detail`,
  `instance` and `operationId`. Quote `operationId` when escalating. A `400` may carry an
  `errors` object naming the offending fields.
- **Do not poll** for changes. Subscribe to `matter.created`, `matter.updated` and
  `matter.status.updated` instead — see `asyncapi/lawvu-webhooks.yml`.
