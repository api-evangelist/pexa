---
name: Discharge a mortgage through PEXA
description: Run a consolidated standalone mortgage discharge from a financial institution, or create a discharge document inside an existing Exchange workspace.
api: openapi/pexa-standalone-discharge-experience-api-openapi.yaml
operations:
  - processDischarge
  - createUsingPOST
  - retrieveLodgedDocumentsV1UsingGET
  - retrieveWorkspaceSummaryV2UsingGET
generated: '2026-07-26'
method: generated
source: openapi/pexa-standalone-discharge-experience-api-openapi.yaml + openapi/pexa-exchange-api-swagger.json
---

# Discharge a mortgage through PEXA

There are two distinct routes, and they are not interchangeable.

## Route A — standalone discharge (financial institutions)

Use the **Standalone Discharge Experience API** when a mortgage is being discharged on its own,
outside a full transfer workspace. It is an OpenAPI 3.1.0 *experience* API with exactly one
operation:

- `processDischarge` — `POST /v1/discharge`, tagged `Standalone Discharge Experience`.
  Servers: `https://api.pexa.com.au` (production) and `https://api-tst.pexalabs.com.au` (test).
  Security: OAuth 2.0 `authorizationCode` and `clientCredentials`.

This is a single composite call that orchestrates the discharge behind PEXA's own service boundary.
It declares only `200` and `400` — a `400` means the request itself was rejected, so read the
response body rather than retrying. **It has no idempotency key and it moves a real dealing against a
real title: never auto-retry it.** On an ambiguous timeout, confirm state out of band before calling
again.

## Route B — a discharge document inside an existing workspace

When the discharge is one dealing inside a broader settlement, work through the Exchange API instead:

1. `createUsingPOST` — `POST /v2/workspace/documents/dischargeMortgage` creates the discharge
   document in the workspace.
2. `retrieveLodgedDocumentsV1UsingGET` — `GET /v2/workspace/documents` reads the lodged documents.
3. `retrieveWorkspaceSummaryV2UsingGET` — `GET /v5/workspace` confirms the workspace state, including
   document-level `lodgementVerification` and (from release R.24.01.00) the
   `verificationResultDetails` array with `complianceLevel` of ERROR / WARNING / INFORMATION.

Then follow the settlement path in the `pexa-create-workspace-and-settle` skill.

## Events to watch

Register for these on the Notification Service (see `pexa-subscribe-to-workspace-events`):
`DISCHARGE_AUTHORITY_UPLOADED`, `DOCUMENT_CREATED`, `DOCUMENT_SIGNED`,
`LODGEMENT_VERIFICATION_WITH_ERRORS`, `LODGEMENT_VERIFICATION_WITH_WARNINGS`, `LODGEMENT_LODGED`,
`LODGEMENT_REGISTERED`, `LOAN_PAYOUT_LINE_ITEM_CREATED`, `LOAN_PAYOUT_LINE_ITEM_UPDATED`,
`SETTLEMENT_SUCCEEDED`. All are in the catalogue at `asyncapi/pexa-notification-webhooks.yml`.

## Rules an agent must follow

- Signing a discharge document and authorising the payout are consequential acts against real money
  and a real land title. Classify them as human-in-the-loop — see
  `agentic-access/pexa-agentic-access.yml` — and hand the user into PEXA Web via the documented deep
  links in `components/pexa-components.yml`.
- The Standalone Discharge API is an *experience* API: it fans out to services you cannot see. Do not
  assume you can reconstruct or reverse its effect through the Exchange operations.
