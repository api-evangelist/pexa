# PEXA (pexa)

PEXA (Property Exchange Australia) operates Australia's dominant Electronic Lodgement Network (ELN), the digital rail on which property is settled and title dealings are lodged with the state land registries. Where the Australian real estate value chain splits into a listings duopoly (REA Group and Domain) sitting over progressively privatised state land registries, PEXA occupies the transaction layer underneath both — electronic conveyancing, settlement funds movement, digital signing and lodgement — and e-conveyancing is now effectively mandated for most dealings in most Australian jurisdictions, making PEXA the closest thing in the property sector to a required, national, machine-readable rail. Its API posture is correspondingly unusual: PEXA publishes a genuine developer portal at developer.pexa.com.au with Swagger UI reference documentation and machine-readable OpenAPI/Swagger contracts for the Exchange, Projects, Notification (webhooks), Standalone Discharge and PEXA Plus Marketplace APIs, and the ARNECC Model Operating Requirements oblige it to offer API access to third parties on an equivalent basis. But that surface is not self-serve: PEXA states plainly that access to the Developer Portal and to test or production credentials is contingent on having signed PEXA's API Agreement, so a developer registers via a form, is validated and approved, signs an agreement, and is then issued OAuth 2.0 client credentials (or mutual TLS). RESO is absent — it is a North American MLS standard with no bearing on Australian e-conveyancing — so there is no RESO Web API or Data Dictionary certification here, and none should be expected. This is a well-documented, regulated, licensed-access API estate rather than an open one.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/pexa/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/pexa/refs/heads/main/apis.yml)

## Tags

- Real Estate
- Australia
- Conveyancing
- Property Settlement
- Land Registry
- Title
- PropTech
- Mortgage
- Digital Signing
- Webhooks

## Timestamps

- **Created:** 2026-07-26
- **Modified:** 2026-07-26

## APIs

### PEXA Exchange API

The PEXA Exchange API covers key facets of PEXA Exchange e-conveyancing functionality — creating and updating workspaces, invitations, participants, land title references, documents, conversations, financial settlement line items, settlement dates and settlement completion records. Documented as a Swagger 2.0 contract across versioned paths (/v2 through /v6). Authenticated with OAuth 2.0 (client credentials or authorization code) or mutual TLS; each endpoint requires specific scopes. Access requires a signed PEXA API Agreement.

- **Human URL:** [https://developer.pexa.com.au/Exchange/api/apireference/](https://developer.pexa.com.au/Exchange/api/apireference/)
- **Base URL:** `https://api.pexa.com.au`

#### Tags

- Conveyancing
- Property Settlement
- Workspace
- Land Registry
- Title

#### Properties

- [Swagger](openapi/pexa-exchange-api-swagger.json)
- [Swagger](openapi/pexa-exchange-api-legacy-swagger.json)
- [API Reference](https://developer.pexa.com.au/Exchange/api/apireference/)
- [Documentation](https://developer.pexa.com.au/Exchange/docs/documentation/)
- [Pricing](https://www.pexa.com.au/api-pricing/)

### PEXA Projects API

The PEXA Projects API (v4) supports multi-lot development settlements — creating and managing projects, project lots, project statuses and related participants for property developers settling many lots through PEXA. Published as an OpenAPI 3.0.3 contract with OAuth 2.0 authorizationCode and clientCredentials flows and named scopes such as `au:projects:project:write`.

- **Human URL:** [https://developer.pexa.com.au/Projects/api/apireference/](https://developer.pexa.com.au/Projects/api/apireference/)
- **Base URL:** `https://api.pexa.com.au`

#### Tags

- Project
- Property Development
- Conveyancing

#### Properties

- [OpenAPI](openapi/pexa-projects-api-v4-openapi.yaml)
- [API Reference](https://developer.pexa.com.au/Projects/api/apireference/)
- [Documentation](https://developer.pexa.com.au/Projects/docs/documentation/)

### PEXA Notification Service API

The PEXA Push Notifications Service delivers event notifications to subscribers via webhook and manages registration for specific events, including creating, updating and deleting notification registrations and rotating the shared secret used to sign webhook deliveries. Published as an OpenAPI 3.1.0 contract that declares two outbound webhooks — `SubscriberNotification` and `NonSubscriberNotification`. PEXA documents this as a premium, billed API.

- **Human URL:** [https://developer.pexa.com.au/Webhooks/api/apireference/](https://developer.pexa.com.au/Webhooks/api/apireference/)
- **Base URL:** `https://api.pexa.com.au`

#### Tags

- Webhooks
- Notifications
- Events

#### Properties

- [OpenAPI](openapi/pexa-notification-service-openapi.yaml)
- [API Reference](https://developer.pexa.com.au/Webhooks/api/apireference/)

### PEXA Standalone Discharge Experience API

An OpenAPI 3.1.0 experience API supporting consolidated standalone mortgage discharge, used by financial institutions discharging a mortgage outside of a full transfer workspace. Declares production and test servers and OAuth 2.0 authorizationCode and clientCredentials flows.

- **Human URL:** [https://developer.pexa.com.au/EXP_ConsolidatedDischarge/api/apireference/](https://developer.pexa.com.au/EXP_ConsolidatedDischarge/api/apireference/)
- **Base URL:** `https://api.pexa.com.au`

#### Tags

- Mortgage
- Discharge
- Financial Institutions

#### Properties

- [OpenAPI](openapi/pexa-standalone-discharge-experience-api-openapi.yaml)
- [API Reference](https://developer.pexa.com.au/EXP_ConsolidatedDischarge/api/apireference/)

### PEXA Plus Marketplace B2B API

The PEXA Plus Marketplace B2B API exposes marketplace services to business partners — health check, title search ordering and billing — under the PEXA Plus product. Published as an OpenAPI 3.0.3 contract (an OpenAPI 3.0.0 variant of the same document is also shipped in the portal) with a documented server at `https://plus.pexa.com.au/api/plus/v1`.

- **Human URL:** [https://developer.pexa.com.au/Marketplace/api/apireference/](https://developer.pexa.com.au/Marketplace/api/apireference/)
- **Base URL:** `https://plus.pexa.com.au/api/plus/v1`

#### Tags

- Marketplace
- Title Search
- Billing

#### Properties

- [OpenAPI](openapi/pexa-plus-marketplace-b2b-api-openapi.yaml)
- [OpenAPI](openapi/pexa-plus-marketplace-b2b-api-oas300-openapi.yaml)
- [API Reference](https://developer.pexa.com.au/Marketplace/api/apireference/)

## Common Properties

- [Website](https://www.pexa.com.au/)
- [Portal](https://developer.pexa.com.au/)
- [Documentation](https://developer.pexa.com.au/)
- [Sign Up](https://www.pexa.com.au/pexa-apis/)
- [Pricing](https://www.pexa.com.au/api-pricing/)
- [Documentation — Integration Principles (PDF)](https://www.pexa.com.au/staticly-media/2024/10/PEXA_Integration_Principles-sm-1728949058.pdf)
- [Authentication — OpenID Connect discovery](https://auth.pexa.com.au/.well-known/openid-configuration)
- [Status Page](https://status.pexa.com.au/)
- [Support](https://www.pexa.com.au/contact-us/)
- [Blog](https://www.pexa-group.com/content-hub/news/)
- [About](https://www.pexa-group.com/about/)
- [Investor Relations](https://www.pexa-group.com/investor-centre/)

## Access

- **Home market:** Australia
- **Access gate:** `licence-agreement` — "Access to the Developer Portal and credentials to test and productionise APIs is contingent upon having signed PEXA's API Agreement." Registration is by form on [pexa.com.au/pexa-apis/](https://www.pexa.com.au/pexa-apis/), validated and approved by PEXA, with credentials emailed afterwards. There is no self-serve signup.
- **Auth model:** OAuth 2.0 (client credentials and authorization code) or mutual TLS. Production token endpoint `https://auth.pexa.com.au/oauth/token`; test token endpoint `https://auth-tst.pexalabs.com.au/oauth/token`. Tokens expire after 12 hours. Anonymous OIDC discovery is served.
- **Sandbox:** Yes — a fully managed test environment at `api-tst.pexalabs.com.au`, with credentials issued after approval.
- **RESO posture:** No RESO reference found. RESO is a North American MLS standard and does not apply to Australian electronic conveyancing; PEXA is not RESO certified and makes no such claim. Australia's machine-readable property mandate is regulatory (ARNECC Model Operating Requirements), not standards-body driven.
- **Open data:** No. Every documented endpoint is behind OAuth 2.0 or mutual TLS and a signed agreement.

## Maintainers

- Kin Lane — kin@apievangelist.com
