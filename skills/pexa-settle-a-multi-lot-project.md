---
name: Settle a multi-lot development through PEXA Projects
description: Create a PEXA project, add a workspace per lot, attach parties and participants, load financial line items and propose the settlement date across the development.
api: openapi/pexa-projects-api-v4-openapi.yaml
operations:
  - createProject
  - updateProject
  - statuses
  - workspaces
  - createWorkspace
  - updateWorkspace
  - addParties
  - updateParties
  - deleteParties
  - addParticipants
  - deleteParticipants
  - createFinancials
  - proposeSettlement
  - notification
generated: '2026-07-26'
method: generated
source: openapi/pexa-projects-api-v4-openapi.yaml + conventions/pexa-conventions.yml
---

# Settle a multi-lot development through PEXA Projects

PEXA Projects is the developer-side product: one **project** groups many lot **workspaces** so a
property developer can settle a whole development without driving each workspace by hand. The
contract is OpenAPI 3.0.3 on `/v4`, secured with OAuth 2.0 (`authorizationCode` and
`clientCredentials`) using `au:projects:*` scopes.

## Scopes

| Scope | Grants |
|---|---|
| `au:projects:project:write` | create and update projects |
| `au:projects:status:read` | read project statuses |
| `au:projects:workspaces:read` | read project workspaces |
| `au:projects:workspaces:write` | create and update project workspaces |
| `au:projects:notification:read` | read project notifications |

The shipped document leaves `servers[0].url` as the unresolved template `{{ service.url }}`. Use
`https://api.pexa.com.au` (production) or `https://api-tst.pexalabs.com.au` (test) — the hosts PEXA
documents on the portal.

## Steps

1. **Create the project** — `createProject` (`POST /v4/projects`, scope
   `au:projects:project:write`). Keep the returned `projectId`.
2. **Add a workspace per lot** — `createWorkspace`
   (`POST /v4/projects/{projectId}/workspaces`). Amend with `updateWorkspace`
   (`PUT /v4/projects/{projectId}/workspaces/{pexaWorkspaceId}`).
3. **Attach the parties** — `addParties`
   (`POST /v4/projects/{projectId}/workspaces/{workspaceId}/parties`), then `updateParties` (PUT) and
   `deleteParties` (DELETE) on the same path.
4. **Attach the participants** — `addParticipants`
   (`POST /v4/projects/{projectId}/workspaces/{workspaceId}/participants`) and `deleteParticipants`
   (DELETE) — the subscribers who will act in each lot workspace.
5. **Load the money** — `createFinancials`
   (`POST /v4/projects/{projectId}/workspaces/{workspaceId}/financials`) creates the financial
   settlement line items for that lot.
6. **Propose settlement** — `proposeSettlement`
   (`POST /v4/projects/{projectId}/workspaces/{pexaWorkspaceId}/settlementDates`).
7. **Track it** — `statuses` (`GET /v4/projects/{projectId}/statuses`) and `workspaces`
   (`GET /v4/projects/{projectId}/workspaces`); `notification`
   (`GET /v4/projects/notifications`) polls project notifications.

## Rules an agent must follow

- **Two different path parameter names mean two different things.** `workspaceId` is used on the
  parties / participants / financials paths; `pexaWorkspaceId` is used on the workspace update and
  settlement-date paths. Do not interchange them.
- **No idempotency key.** Repeating `createWorkspace` or `addParties` after a timeout duplicates
  records. Re-read with `workspaces` first.
- **Deletes are unrecoverable through the API.** `deleteParties` and `deleteParticipants` change who
  can act on a settlement — confirm with a human before calling either.
- Projects settlements ultimately run through PEXA Exchange, so the Exchange rules in
  `pexa-create-workspace-and-settle` (signing is a human act, coded exceptions, per-operation
  versioning) all still apply.
