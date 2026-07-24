---
name: Bulk-export FHIR data
description: Kick off and collect a FHIR Bulk Data ($export) job from an Aidbox box.
api: fhir/aidbox-capabilitystatement.json
operations: [start-patient-level-export]
---

# Bulk-export FHIR data (Aidbox)

Aidbox implements the HL7 FHIR Bulk Data Access IG (its CapabilityStatement instantiates
`http://hl7.org/fhir/uv/bulkdata/CapabilityStatement/bulk-data`).

## Auth
SMART Backend Services token: `client_credentials` + `private_key_jwt`, with a
`system/*.read` (or narrower `system/{Type}.read`) scope. See `authentication/aidbox-authentication.yml`.

## Steps
1. **Kick off** the export (async).
   - Patient-level: `GET <base>/fhir/Patient/$export`
   - System-level: `GET <base>/fhir/$export` (system-level export supported since 2606)
   - Send `Accept: application/fhir+json` and `Prefer: respond-async`.
   - Narrow with `_type=Patient,Observation` and trim columns with `_elements` (2606+).
   - The `202 Accepted` response returns a `Content-Location` polling URL.
2. **Poll** the `Content-Location` URL until `200`; `202` with `X-Progress` means still running.
3. **Download** each NDJSON file listed in the completion manifest's `output[]`.
4. **Clean up** by DELETEing the polling URL when done.

## Rules
- Consent-aware export (opt-in/opt-out filtering, 2605) and de-identified SQL-on-FHIR
  exports (HIPAA Safe Harbor, 2604) are available for compliant data sharing.
- Errors surface as `OperationOutcome` (`errors/aidbox-problem-types.yml`).
