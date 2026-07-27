---
name: Subscribe to PEXA workspace events over webhooks
description: Register a webhook endpoint with the PEXA Notification Service, verify HMAC signatures, dedupe on webhook-id, and rotate the shared secret before it expires.
api: openapi/pexa-notification-service-openapi.yaml
operations:
  - createNotificationRegistrationV3
  - createNotificationRegistration
  - getNotificationRegistrationsV2
  - updateNotificationRegistrationV3
  - deleteNotificationRegistration
  - createNotificationSecretRotation
generated: '2026-07-26'
method: generated
source: openapi/pexa-notification-service-openapi.yaml + asyncapi/pexa-notification-webhooks.yml
---

# Subscribe to PEXA workspace events over webhooks

The PEXA Push Notifications Service delivers real-time workspace events to an HTTPS endpoint you
control. It is a **premium, billed** API — contact `apisupport@pexa.com.au`. The full event catalogue
(110 event types), retry schedule and signature contract live in
`asyncapi/pexa-notification-webhooks.yml`.

## Before you start

- Your webhook domain must be **whitelisted by PEXA during onboarding**. A registration whose
  `webhookURI` domain does not match the whitelisted domain fails with `GA.NOTIF.400014`.
- All operations use OAuth 2.0 **client credentials** against
  `https://auth.pexa.com.au/oauth/token`, each with its own scope.

## Steps

1. **Register** — `createNotificationRegistrationV3` (`POST /v3/notification-registrations`,
   scope `create:notification_registrations`). Body: `notificationType: WEBHOOK`, `eventTypes[]`,
   `attributes.webhookURI`, `attributes.authenticationMode` (`HMAC` or `HMAC_OAUTH`) and, for
   `HMAC_OAUTH`, `attributes.authConfig` (`authUrl`, `clientId`, `clientAuthMethod` of
   `private_key_jwt` or `client_secret`, optional `audience` and `scope`).
   Use `createNotificationRegistration` (`POST /v1/notification-registrations`) only for plain HMAC.
   **Call this exactly once per endpoint.** There is no idempotency key: a duplicate call creates a
   second identical registration and you will receive every event twice (`GA.NOTIF.400019` catches
   only exact duplicates).
2. **Store the response** — `registrationId` (needed for every later change), `sharedSecret` (put it
   in a secrets manager, never in code or base64) and `sharedSecretExpiry`.
3. **Include the mandatory event** — `SECRET_EXPIRY` must be in `eventTypes` (`GA.NOTIF.400020`) and
   at least one other event must accompany it (`GA.NOTIF.400021`).
4. **Verify every delivery.** Compute `HMAC-SHA256(sharedSecret, "webhook-id.webhook-timestamp.payload")`,
   hex encode it, prefix `v1,` and compare against the `webhook-signature` header. The header carries
   a space-delimited list — during rotation it holds signatures from both the new and old secret for
   24 hours, so accept a match against any entry.
5. **Dedupe on `webhook-id`.** PEXA documents `webhook-id` as the idempotency key: it is stable across
   every retry of the same event. Persist seen ids and drop repeats before doing any work.
6. **Acknowledge within 5 seconds.** Any 2xx ends delivery. **3xx and 4xx are hard failures with no
   retry** — if you return one by mistake the event is lost and PEXA must re-trigger it manually. Only
   500/502/503/504 are retried, on the schedule 5s, 5m, 30m, 2h, 5h, 10h. Queue the payload and
   process asynchronously; never do downstream work inside the acknowledgement path.
7. **Rotate before expiry** — the shared secret dies after **30 days** and all registrations are
   deleted on expiry. You will receive `SECRET_EXPIRY` 7 days out and daily thereafter. On that event
   call `createNotificationSecretRotation`
   (`POST /v1/notification-registrations/{registrationId}/secret-rotation`, scope
   `create:notification_registrations_secret_rotation`) and accept both the old and new signature for
   24 hours. Automate this — it is the single most common way a PEXA webhook integration dies.
8. **Audit and amend** — list registrations with `getNotificationRegistrationsV2`
   (`GET /v2/notification-registrations`, scope `view:notification_registrations`, paged via `page`
   and `limit` 1-20). Change an endpoint or event set with `updateNotificationRegistrationV3`
   (`PUT /v3/notification-registrations/{registrationId}`, scope `edit:notification_registrations`)
   and stop delivery with `deleteNotificationRegistration`
   (`DELETE /v1/notification-registrations/{registrationId}`, scope
   `delete:notification_registrations`, returns 204).

## Reading a payload

`type` is `<EventCategory>.<EventName>` — e.g. `Lodgement.LODGEMENT_REGISTERED`,
`Workspace.CHECKLIST_CHECKED`. `data.workspaceId` is the PEXA workspace id you use to call back into
the Exchange API, `data.subscriberReferences[]` carries your own reference and the workspace role,
and `data.links[]` is a HATEOAS list of the Exchange operations relevant to this event. PEXA states
the payload carries **no PII**, so treat it as a trigger to fetch, not as the record itself.

## HMAC_OAUTH mode

In `HMAC_OAUTH`, PEXA authenticates itself to **your** token endpoint before each delivery and sends
`Authorization: Bearer <token>` alongside the HMAC signature. Validate both. With `private_key_jwt`,
PEXA signs the client assertion with an AWS KMS RSA key using PS256 — import PEXA's public key into
your auth server (request it from `apisupport@pexa.com.au`). Optional mTLS is available in this mode
only, and applies to both the token request and the webhook delivery.
