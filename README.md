# Aidbox

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Aidbox is a production-ready FHIR platform and clinical data backend from Health Samurai, used by digital health startups, providers, payers, and life-sciences teams to build interoperable healthcare products in the United States and globally. Built on PostgreSQL, Aidbox exposes a full HL7 FHIR REST API (R4, R5, and R6) plus Aidbox-native SQL search, SQL-on-FHIR, a GraphQL API, Bulk Data export/import, topic-based Subscriptions, and terminology services. It is self-hosted or Aidbox-hosted software rather than a single public multi-tenant endpoint, so each deployment serves its own FHIR base and its own CapabilityStatement; a public sandbox (sandbox.aidbox.app) exposes a live FHIR R4 CapabilityStatement and a SMART-on-FHIR / OpenID .well-known configuration. Authentication is OAuth 2.0 with SMART-on-FHIR support (patient/user/system scopes, EHR and standalone launch, client-confidential and client-public flows). Aidbox aligns with US interoperability needs (US Core, SMART App Launch, Bulk Data) and is HIPAA-oriented, HITRUST and SOC 2 focused. This profile catalogs the documented, real API surface only.

**Areas:** Healthcare · United States · FHIR · HL7 · Interoperability · SMART on FHIR · Electronic Health Records · Health Data · FHIR Server · Bulk Data · Terminology · Digital Health

## Aidbox on APIs.io

Everything profiled for Aidbox is published on APIs.io rather than in this repository. Start at the provider page — **[apis.io/providers/aidbox/](https://apis.io/providers/aidbox/)** — or go straight to one of the APIs or properties below.

### APIs (7)

- [Aidbox FHIR API](https://apis.io/apis/aidbox/aidbox-fhir-api/) — Aidbox's HL7 FHIR REST API for creating, reading, updating, deleting, and searching clinical and administrative resources. Supports FHIR R4 (4.0.1), R5, and…
- [Aidbox REST & Search API](https://apis.io/apis/aidbox/aidbox-rest-search-api/) — Aidbox-native REST and search API offering CRUD over resources plus SQL-based search implementations for performance optimization and complex custom queries…
- [Aidbox SQL-on-FHIR API](https://apis.io/apis/aidbox/aidbox-sql-on-fhir-api/) — SQL-on-FHIR support that lets clients run SQL queries directly against FHIR data, treating FHIR resources as relational tables for analytics and reporting.
- [Aidbox GraphQL API](https://apis.io/apis/aidbox/aidbox-graphql-api/) — A GraphQL API over FHIR data that lets clients request exactly the data they need across related resources in a single call.
- [Aidbox Bulk Data API](https://apis.io/apis/aidbox/aidbox-bulk-data-api/) — FHIR Bulk Data API for large-scale export and import of FHIR resources, including $export, FHIR import/load, and dump utilities for moving datasets in and…
- [Aidbox Topic-Based Subscriptions API](https://apis.io/apis/aidbox/aidbox-subscriptions-api/) — FHIR topic-based Subscriptions that deliver notifications on changes to FHIR resources through multiple channels for event-driven healthcare integrations.
- [Aidbox Terminology API](https://apis.io/apis/aidbox/aidbox-terminology-api/) — Terminology services for working with FHIR CodeSystems, ValueSets, and ConceptMaps, including validation and lookup against clinical terminologies.

### Published properties

Each property below is a page on APIs.io carrying the machine-readable artifact and what it says.

#### AsyncAPI (1)

Event-driven and webhook contracts.

- [Aidbox Subscriptions Webhooks](https://apis.io/asyncapis/aidbox/aidbox-subscriptions-webhooks/)

#### MCP servers (1)

Model Context Protocol servers that expose the API to agents.

- [aidbox-mcp](https://apis.io/servers/aidbox/aidbox-mcp/)

#### Agent skills (1)

Agent skills built on the API.

- [aidbox](https://apis.io/agent-skills/aidbox/aidbox/)

#### Security (2)

Authentication, trust centre, and vulnerability disclosure.

- [Aidbox Authentication](https://apis.io/security/aidbox/aidbox-authentication/)
- [Aidbox Domain Security](https://apis.io/security/aidbox/aidbox-domain-security/)

#### OAuth scopes (1)

The permissions the API asks for.

- [Aidbox Scopes](https://apis.io/scopes/aidbox/aidbox-scopes/)

#### Plans and pricing (1)

Published plans, tiers and prices.

- [Aidbox Plans Pricing](https://apis.io/plans/aidbox/aidbox-plans-pricing/)

#### Rate limits (1)

Published request limits.

- [Aidbox Rate Limits](https://apis.io/rate-limits/aidbox/aidbox-rate-limits/)

## Where to find this API

Links to the company's own pages. This repository publishes no copy of their specifications — the canonical document is theirs and lives at the URLs below.

- [Website](https://www.health-samurai.io/)
- [Documentation](https://www.health-samurai.io/docs/aidbox)
- [Getting started](https://www.health-samurai.io/docs/aidbox/getting-started)
- [API reference](https://www.health-samurai.io/docs/aidbox/api-1/api)
- [Pricing](https://www.health-samurai.io/docs/aidbox/getting-started/editions-and-pricing)
- [Sign up](https://aidbox.app/)
- [Support](https://www.health-samurai.io/contacts)
- [Blog](https://www.health-samurai.io/blog)
- [Security](https://www.health-samurai.io/docs/aidbox/modules/security-and-access-control)
- [Compliance](https://www.health-samurai.io/fhir-server)
- [Privacy policy](https://www.health-samurai.io/legal/privacy-policy)
- [GitHub](https://github.com/Aidbox)
- [Status](https://status.aidbox.app/)

## API Evangelist rating

**Kin Score 57.8** — strong · rubric 0.25.0 · scored 2026-10-23

| Facet | Score |
|---|---|
| Discoverability | 76.7 |
| Contract quality | 44.2 |
| Contract governance | 18.2 |
| Operational transparency | 60.5 |
| Developer ergonomics | 78.6 |
| Access clarity | 42.1 |

**Agent readiness 41.6** — agent-ready (gated down from agent-native)

| Agent-readiness dimension | Result |
|---|---|
| Spec Presence | yes |
| Agentic Access | — |
| Reversibility Documented | — |
| MCP Server | documented (partial credit) |
| Auth Clarity | served (partial credit) |
| Idempotency | documented (partial credit) |
| Error Semantics | derived (partial credit) |
| OpenAPI Examples | — |
| Rate Limit Signal | documented (partial credit) |
| Event Surface Described | yes |
| Agent Skills | yes |
| Well Known Catalog | — |
| Consent Identity | — |
| Agent Card | — |
| Dry Run Mode | — |
| Delegated Identity | served (partial credit) |
| Protected Resource Metadata | — |
| Dynamic Client Registration | — |
| Agentic Commerce | — |
| Pagination Ratio | na |
| Operation Distinctiveness | na |

**Regulatory alignment 44.5** — Health

The rubric is published at [github.com/api-evangelist/kin-score](https://github.com/api-evangelist/kin-score). The per-check breakdown behind these numbers — which checks were earned, partial or missed — along with score history and how this provider compares to its peers, is available at [apis.io/providers/aidbox/](https://apis.io/providers/aidbox/).

## How this profile is made

API Evangelist profiles a company's public API surface the way a developer would find it: the website, developer portal and documentation, the specifications it publishes, its public repositories, and its pricing, status and changelog pages. From that it derives the properties above — contracts split per API, workflows, schemas, collections, governance rules, plans and rate limits — and scores them against the published [Kin Score rubric](https://github.com/api-evangelist/kin-score).

The artifacts are published on [APIs.io](https://apis.io), not in this repository. This repository is the public record for Aidbox: the place to say something is wrong, missing or out of date, and to ask for a re-score or a removal.

## Open an issue

- **Something about this listing** — a wrong description, a missing API, a stale artifact or a question about the rating: [open an issue on this repository](https://github.com/api-evangelist/aidbox/issues/new).
- **A new API, or one that is not in the catalog yet** — [submit it on APIs.io](https://apis.io/add/).
- **Anything else** — [the APIs.io inbox](https://github.com/api-search/inbox/issues/new) or [info@apievangelist.com](mailto:info@apievangelist.com).
