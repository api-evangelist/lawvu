---
name: Move a LawVu contract through its lifecycle
description: Create a contract, attach and version its document, advance its stage, and retrieve the executed copy.
api: openapi/lawvu-api-openapi-original.yml
operations:
  - get-v2-contracttypes
  - post-v2-contracts
  - post-v2-files
  - get-v2-contracts-contractid
  - patch-v2-contracts-contractid
  - post-v2-files-fileid
  - get-v2-contracts-contractid-files-fileid
  - get-v2-files-fileid-content
---

# Move a LawVu contract through its lifecycle

Use this skill to drive contract lifecycle management from an external system — intake, drafting,
approval, signature and execution.

## Before you start

- OAuth 2.0 bearer token, delegated to the consenting user. See
  `authentication/lawvu-authentication.yml`.
- Contract stages are customer-configured. Read the current `status` before you try to advance
  one; do not assume a fixed stage vocabulary.

## Steps

1. **Pick the contract type.** Call `get-v2-contracttypes`. Note `hasWizard` and
   `teamAssignmentRequired` — when `teamAssignmentRequired` is true, a create without
   `teamAssigned` will fail validation with a `400`.

2. **Create the contract.** Call `post-v2-contracts` with `type` and `name`, plus `owner`,
   `expiry`, `externalId`, `teamAssigned`, `restricted` and custom `fields` as available. The
   response carries the new `id`.

3. **Attach the contract document.** Call `post-v2-files` with `targetResourceType` set to the
   contract resource type and `targetResourceId` set to the contract id.

4. **Version the document as it is negotiated.** Call `post-v2-files-fileid` against the existing
   file id to upload a new version rather than creating a second file — this keeps the version
   history on one record.

5. **Advance the stage.** Call `patch-v2-contracts-contractid` with the new `status`. Send only
   what changed. A `409` means the change conflicts with the contract's current state — re-read
   with `get-v2-contracts-contractid` before retrying.

6. **Record execution.** When signed, `patch-v2-contracts-contractid` with the executed date and
   any post-signature field values.

7. **Retrieve the executed copy.** Use `get-v2-contracts-contractid` to find the `document`
   reference, then `get-v2-contracts-contractid-files-fileid` for its metadata and
   `get-v2-files-fileid-content` to download the bytes.

## Conventions that apply

- **List and filter contracts** with the OData subset: `Id`, `Name` (`contains()` supported),
  `Type/Id`, `Owner/Id`, `ExternalId`, `Restricted`, `Status`, `TeamAssigned/Id`,
  `Created/User/Id`. Sortable: `Id`, `Name`, `Owner/Id`, `TeamAssigned/Id`.
- **No idempotency key exists.** A retried `post-v2-contracts` creates a duplicate contract.
  Guard creates by first querying `$filter=ExternalId eq '{your-id}'`.
- **Errors are RFC 9457.** `400` for validation (check the `errors` extension), `403` for
  permission, `409` for state conflicts, `415` for a wrong upload content type.
- **Subscribe, don't poll.** `contract.created`, `contract.updated`, `contract.status.updated`,
  `contract.document.updated`, `contract.file.created`, `contract.keydate.created` and
  `contract.keydate.updated` cover this whole flow. See `asyncapi/lawvu-webhooks.yml`.
  Note `contract.updated` is deduplicated over a 2-minute delay window.
