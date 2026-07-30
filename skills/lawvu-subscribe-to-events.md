---
name: Subscribe to LawVu events
description: Register a webhook subscription, receive and acknowledge notifications, and hydrate each event into full records.
api: openapi/lawvu-account-openapi-original.json
operations:
  - post-v1-webhooks-subscriptions
  - GetWebhookSubscriptions
  - GetWebhookSubscription
  - DeleteWebHookSubscription
  - get-v2-matters-matterid
  - get-v2-contracts-contractid
  - get-v2-files-fileid
---

# Subscribe to LawVu events

LawVu pushes near-real-time notifications so integrations do not have to poll. Use this skill to
set up and operate that event feed.

## Before you start

- Subscriptions are created **under the security context of an individual user**, not an
  organisation. Use a nominated service user with the right access level — notifications are only
  generated for resources that user can see. Plan for account expiry and for restricted items
  being silently excluded.
- Your receiving endpoint must be **HTTPS and publicly accessible**. LawVu does not support
  endpoints that require authentication, and publishes no IP allowlist.

## Steps

1. **Create the subscription.** `POST /account-apis/v1/webhooks/subscriptions`
   (`post-v1-webhooks-subscriptions`) with:
   - `topics` — the events you want (see the catalog in `asyncapi/lawvu-webhooks.yml`).
   - `subscriptionUrl` — HTTPS, max 2048 characters. Query-string parameters are allowed and
     survive delivery, so use them to route or identify tenants.
   - `clientState` — a secret you generate. It is echoed on every notification; validate it on
     each inbound request and reject anything that does not match. This is the only
     authenticity signal LawVu provides — there is no signature header.

2. **Acknowledge fast.** Respond `2xx` within **5 seconds**, before doing any real work. Queue
   the payload and process it asynchronously. A slow handler triggers redelivery and duplicates.

3. **Hydrate the event.** Notification bodies are thin: `Timestamp`, `ResourceId`, `EventType`,
   `ClientState`, and `ParentResource` on sub-resource topics. Fetch the detail yourself —
   `get-v2-matters-matterid`, `get-v2-contracts-contractid`, or `get-v2-files-fileid` using
   `ResourceId` (and `ParentResource.ResourceId` for file and status-message events).

4. **Handle duplicates.** Deduplication is per topic, not per subscription. A subscription on both
   `matter.updated` and `matter.status.updated` receives two notifications for one state change.
   Key your processing on `EventType` + `ResourceId` + `Timestamp` and make handlers idempotent.

5. **Audit and repair.** List with `GetWebhookSubscriptions`, inspect one with
   `GetWebhookSubscription`. Subscriptions **cannot be updated** — to change topics or the URL,
   `DeleteWebHookSubscription` and create a new one.

## Failure behaviour to design for

- Redelivery on non-acknowledgement: after 1 minute, then 2 minutes, then 5 minutes, then the
  notification is abandoned.
- **10 abandoned notifications within 24 hours disables the subscription**, and a disabled
  subscription must be recreated. Monitor for a silent feed and alert on it — LawVu does not
  notify you that the subscription was disabled.
- Deduplicated topics (`matter.updated`, `contract.updated`) arrive up to 2 minutes late by
  design.
