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
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Aidbox is a production-ready FHIR platform and clinical data backend from Health Samurai, built on PostgreSQL for high-load medical products and integrations. It is used by digital health startups, healthcare providers, payers, and life-sciences teams — with the United States as a primary market — to build interoperable healthcare applications on HL7 FHIR.

Aidbox is self-hosted or Aidbox-hosted software rather than a single public multi-tenant API, so every deployment serves its own FHIR base URL and its own FHIR CapabilityStatement. A public sandbox at `sandbox.aidbox.app` exposes a live FHIR R4 (4.0.1) CapabilityStatement (138 supported resource types) and a SMART-on-FHIR / OpenID `.well-known` configuration.

## APIs

- **Aidbox FHIR API** — HL7 FHIR REST API (R4/R5/R6): CRUD, search, transaction/batch bundles, and operations ($validate, $everything, $document, $lastn).
- **Aidbox REST & Search API** — Aidbox-native REST plus SQL-based search for performance-optimized, complex custom queries.
- **Aidbox SQL-on-FHIR API** — SQL queries directly against FHIR data, treating resources as relational tables.
- **Aidbox GraphQL API** — GraphQL over FHIR data for precise, single-call retrieval across related resources.
- **Aidbox Bulk Data API** — FHIR Bulk Data export/import ($export, import/load, dump utilities).
- **Aidbox Topic-Based Subscriptions API** — event-driven notifications on FHIR resource changes across multiple channels.
- **Aidbox Terminology API** — CodeSystem, ValueSet, and ConceptMap services with validation and lookup.

## Authentication

OAuth 2.0 with SMART-on-FHIR. The sandbox `.well-known/smart-configuration` advertises SMART App Launch capabilities (EHR and standalone launch, client-confidential symmetric/asymmetric, client-public, SSO OpenID Connect) and scopes including `patient/*.cruds`, `user/*.cruds`, `system/*.cruds`, `launch`, `launch/patient`, `fhirUser`, and `offline_access`.

## Links

- Website: https://www.health-samurai.io/
- Documentation / Developer Portal: https://www.health-samurai.io/docs/aidbox
- API Reference: https://www.health-samurai.io/docs/aidbox/api-1/api
- Getting Started: https://www.health-samurai.io/docs/aidbox/getting-started
- Status: https://status.aidbox.app/
- GitHub: https://github.com/Aidbox

Harvested machine-readable contracts are stored under `fhir/`.
