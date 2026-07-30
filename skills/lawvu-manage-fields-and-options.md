---
name: Manage LawVu custom fields and their options
description: Discover custom field definitions and keep option-based field values in sync with an external reference list.
api: openapi/lawvu-api-openapi-original.yml
operations:
  - get-v2-fields
  - get-v2-fields-fieldid-options
  - post-v2-fields-fieldid-options
  - get-v2-fields-fieldid-options-value
  - patch-v2-fields-fieldid-options-value
  - delete-v2-fields-fieldid-options-value
---

# Manage LawVu custom fields and their options

Matters and contracts carry customer-defined custom fields. Use this skill to discover them and
to keep option lists (departments, cost centres, business units) synchronised from an upstream
system.

## Steps

1. **Discover the fields.** Call `get-v2-fields` to list field definitions — `id`, `name`, `type`,
   `properties`. Match by `name`, not by `id`: field ids differ per tenant.

2. **Read the current options.** For an option-based field, call
   `get-v2-fields-fieldid-options` with the field id. This endpoint supports OData: filter on
   `Label` (`eq`, `ne`, `contains()`) and `Value` (`eq`, `ne`), sort by `Label`, and page with
   `$top` / `$skip`. Page through fully before diffing — the default page is 30 items.

3. **Diff against your source list**, then:
   - **Add** — `post-v2-fields-fieldid-options` with the `label` and any `properties`. The
     response carries the assigned `value`, which is the option's stable key.
   - **Update** — `patch-v2-fields-fieldid-options-value` addressed by `value`, to rename a
     `label` or change `properties`.
   - **Remove** — `delete-v2-fields-fieldid-options-value`.

4. **Check one option** before acting on it with `get-v2-fields-fieldid-options-value`; a `404`
   means the option was already removed.

## Notes and cautions

- `value` is the identity of an option; `label` is display text and may change. Store `value` in
  your mapping table.
- Departments support filtering on `Properties.parentId` (`eq`, `ne`, and null forms), so a
  hierarchy can be walked one level at a time.
- Deleting an option that is in use affects existing matters and contracts. Reconcile in
  the sandbox (`https://api-sandbox.lawvu.com`) before running a destructive sync in production.
- `userCanAddOption` in a field's properties tells you whether end users create options too — if
  true, treat your sync as additive and avoid deleting options you did not create.
- Filtering matters and contracts *by* field values uses the `any` collection operator, e.g.
  `$filter=Fields/lookup/any(lookup:lookup/value eq 'option_a')`. Only `or` logic works inside
  `any`.
- Errors follow RFC 9457; a `400` on an OData query names the unsupported field or operator in
  `detail`.
