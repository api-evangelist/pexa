---
name: Order a title search and reconcile billing on PEXA Plus Marketplace
description: Health check, place a title search order, poll for the result, download the returned documents and pull the billing invoice from the PEXA Plus Marketplace B2B API.
api: openapi/pexa-plus-marketplace-b2b-api-openapi.yaml
operations:
  - getHealth
  - createTitleSearch
  - retrieveTitleSearch
  - retrieveTitleSearchDocuments
  - retrieveBilling
generated: '2026-07-26'
method: generated
source: openapi/pexa-plus-marketplace-b2b-api-openapi.yaml + errors/pexa-problem-types.yml
---

# Order a title search and reconcile billing on PEXA Plus Marketplace

The PEXA Plus Marketplace B2B API is the smallest and most self-contained PEXA surface: five
operations over `https://plus.pexa.com.au/api/plus/v1`, covering health, title search and billing.
It is the one contract with its own dedicated production server declared in the spec.

## Steps

1. **Health check** — `getHealth` (`GET /health`). Use it as a readiness probe before a batch of
   orders, not as a per-request preflight.
2. **Place the order** — `createTitleSearch` (`POST /title-search`). Returns `201` with a
   `titleSearchId`. **There is no idempotency key** — persist the `titleSearchId` immediately and
   never resubmit an order after a timeout without first checking whether one already landed. Title
   searches are chargeable.
3. **Poll for the result** — `retrieveTitleSearch` (`GET /title-search/{titleSearchId}`). Back off
   between polls; no rate limit is published and none should be assumed to be generous.
4. **Fetch the documents** — `retrieveTitleSearchDocuments`
   (`GET /title-search/{titleSearchId}/documents`) once the search has completed.
5. **Reconcile** — `retrieveBilling` (`GET /billing`) returns the billing invoice, so an agent can
   tie orders back to charges.

## Error handling

This contract declares `400`, `401` (documented as "Unauthorised"), `403`, `404`, `405` "Method not
allowed" and `500`. Several of those carry no description in the shipped document. The Marketplace
does **not** use the Exchange `OB*` exception registry — treat the HTTP status and the response body
`Error` object as the signal, and see `errors/pexa-problem-types.yml` for the derived response map.

## Notes

- Two documents ship for this API: an OpenAPI 3.0.3 (`pexa-plus-marketplace-b2b-api-openapi.yaml`,
  authoritative) and an OpenAPI 3.0.0 variant of the same `1.0.2` content. Build against the 3.0.3.
- Tags are `HealthCheck`, `TitleSearch` and `Billing`.
- Access is still gated by the PEXA API Agreement. `plus.pexa.com.au` did not answer anonymous
  `/.well-known/` probes.
