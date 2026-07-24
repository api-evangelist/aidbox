---
name: Search FHIR resources
description: Run FHIR searches against an Aidbox box with search parameters, includes, and pagination.
api: fhir/aidbox-capabilitystatement.json
operations: [search-fhir-resources]
mcp_tools: [search-fhir-resources]
---

# Search FHIR resources (Aidbox)

## Auth
Bearer token with a read/search scope for the target type (e.g. `patient/Observation.s`
or `user/*.rs`). See `scopes/aidbox-scopes.yml`.

## Steps
1. **Search** by type + parameters.
   - MCP: call `search-fhir-resources` with `resourceType` and the query params.
   - REST: `GET <base>/fhir/Observation?patient={id}&code=http://loinc.org|1234-5&_sort=-date&_count=50`
2. **Read the searchset Bundle.** The response is a `Bundle` (`type: searchset`) with
   `Bundle.total` and `Bundle.link` (self / next). Follow the `next` link URL verbatim to page.
3. **Widen with graph params** as needed: `_include`, `_revinclude`, `_has`, chained params.

## Rules
- Use `_elements` / `_summary` to trim payloads (see `conventions/aidbox-conventions.yml`).
- For analytics-scale queries, prefer Aidbox SQL search or SQL-on-FHIR ViewDefinition
  rather than deep chained FHIR search.
- Unknown/unsupported search params return an `OperationOutcome` error (since 2606) rather
  than being silently ignored — fix the param, don't retry blindly.
