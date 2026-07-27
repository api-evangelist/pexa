---
name: Create a PEXA workspace and take it to settlement
description: Verify a land title, create an Exchange workspace, invite the other participants, load the financial settlement schedule, agree the settlement date and read the settlement completion record.
api: openapi/pexa-exchange-api-swagger.json
operations:
  - LTRV
  - workspacecreation
  - subscribersearch
  - createInvitationUsingPOST
  - retrieveWorkspaceSummaryV2UsingGET
  - createUsingPOST_1
  - retrieveSetllementStatementV1UsingGET
  - settlementAvailability
  - maintainsettlement_1
  - retrieveSettlementCompletionRecordV1UsingGET
generated: '2026-07-26'
method: generated
source: openapi/pexa-exchange-api-swagger.json + conventions/pexa-conventions.yml + errors/pexa-error-codes.yml
---

# Create a PEXA workspace and take it to settlement

PEXA Exchange is Australia's Electronic Lodgement Network. A **workspace** is the settlement
container: participants join it in a workspace role, land titles are attached, documents are created
and signed, a financial settlement schedule is built and signed, a settlement date and time is
agreed, then lodgement happens against the state Land Registry and funds move.

## Before you start

- You need a **signed PEXA API Agreement**. Credentials are not self-serve — they are issued by
  `apisupport@pexa.com.au` after PEXA validates your registration. See `sandbox/pexa-sandbox.yml`.
- Get an access token from `https://auth.pexa.com.au/oauth/token` (production) or
  `https://auth-tst.pexalabs.com.au/oauth/token` (test) using `grant_type=client_credentials` with
  the scope the endpoint declares. Tokens last **12 hours**. TLS 1.2 or above is mandatory.
- Send it as `Authorization: Bearer <token>`. Full auth model: `authentication/pexa-authentication.yml`.
- **There is no idempotency key on Exchange writes.** Never blind-retry a POST or PUT — a repeat call
  creates a repeat resource. On a timeout, re-read state with `retrieveWorkspaceSummaryV2UsingGET`
  before deciding whether to act again.

## Steps

1. **Verify the land title** — `LTRV` (`GET /v2/landRegistry/titleStatus`). Confirms the title exists
   and is not already sitting in another workspace. `OB1401.006` means the title already exists in a
   PEXA workspace; `OB1401.007` means it was not found — in test, use a title supplied by the PEXA
   API Services team.
2. **Create the workspace** — `workspacecreation` (`POST /v5/workspace`). Returns the PEXA workspace
   id in the form `PEXA<digits>`. This id is the correlation handle for every later call and for
   every webhook event. Store it.
3. **Find the counterparties** — `subscribersearch` (`GET /v4/subscriber`). Pass EITHER the
   subscriber id OR the subscriber name, never both (`OB1400.005`) and never neither (`OB1400.006`).
4. **Invite participants** — `createInvitationUsingPOST` (`POST /v2/workspace/invitation`) per
   participant. If you are the invitee instead, respond with `respondInvitationV1UsingPUT`
   (`PUT /v2/subscriber/invitation`) and list what is outstanding with
   `retrieveOutstandingInvitationsV1UsingGET`.
5. **Poll or subscribe for state** — `retrieveWorkspaceSummaryV2UsingGET` (`GET /v5/workspace`) is the
   canonical read. Prefer webhooks over polling: register for `PARTICIPANT_INVITED`,
   `INVITATION_ACCEPTED`, `WORKSPACE_STATUS_CHANGED` and `SETTLEMENT_*` events instead — see the
   `pexa-subscribe-to-workspace-events` skill.
6. **Load the financial settlement schedule** — `createUsingPOST_1`
   (`POST /v2/workspace/financial`) uploads the settlement line items, then read it back with
   `retrieveSetllementStatementV1UsingGET` (`GET /v2/workspace/financial`). Note the spelling of that
   operationId — it is misspelled in PEXA's contract and must be used verbatim.
7. **Agree the balance due to effect settlement** — `getBalanceDueToEffectSettlementUsingGET` then
   `updateBalanceDueToEffectSettlementUsingPUT` on
   `/v2/workspace/{workspaceId}/settlement/balanceDueToEffectSettlement`.
8. **Book settlement** — check availability with `settlementAvailability`
   (`GET /v2/workspace/settlement`), then set the date and time with `maintainsettlement_1`
   (`PUT /v2/workspace/settlement`).
9. **Attach documents as needed** — `createUsingPOST`
   (`POST /v2/workspace/documents/dischargeMortgage`) to create a discharge document, and
   `retrieveLodgedDocumentsV1UsingGET` (`GET /v2/workspace/documents`) to read lodged documents.
10. **Close the loop** — `retrieveSettlementCompletionRecordV1UsingGET`
    (`GET /v2/workspace/settlementCompletionRecord`) generates the settlement completion record.

## Rules an agent must follow

- **Version numbers are per operation, not per API.** The current Exchange contract serves `/v2`
  through `/v6` in one document. Always take the path verbatim from the spec — do not normalise them.
- **Signing and authorising are human acts.** Document signing, settlement-schedule signing, source
  line-item authorisation and settlement acceptance move real money against real land titles. Route
  them through PEXA Web via the documented deep links in `components/pexa-components.yml` rather than
  attempting to automate consent. See `agentic-access/pexa-agentic-access.yml`.
- **Errors are coded.** Business-rule failures return an `OB<nnnn>.<nnn>` exception code with a terse
  message. Look the code up in `errors/pexa-error-codes.yml` for the cause and the remediation, and
  surface a human-meaningful message rather than the raw code — that is PEXA's own instruction.
- **No published rate limits.** There is no documented quota and no `X-RateLimit-*` header. Metering
  is commercial, per Settled Workspace. Be conservative with polling.
