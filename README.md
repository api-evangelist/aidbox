# Aidbox

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
