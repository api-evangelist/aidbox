---
name: Idempotent conditional upsert
description: Safely create-or-update a FHIR resource by business key using conditional update, so retries never duplicate.
api: fhir/aidbox-capabilitystatement.json
operations: [conditional-update-fhir-resource, validate-fhir-resource]
mcp_tools: [conditional-update-fhir-resource, validate-fhir-resource]
---

# Idempotent conditional upsert (Aidbox)

Use this to ingest records keyed by an external identifier (MRN, order number) exactly
once, even when the write is retried.

## Auth
Bearer token with create+update scope on the target type (e.g. `system/Patient.cu`).

## Steps
1. **(Optional) Validate** the resource first.
   - MCP: `validate-fhir-resource` (available since v2509).
   - REST: `POST <base>/fhir/{Type}/$validate` — a `422` OperationOutcome means fix before writing.
2. **Conditional update** against the business key.
   - MCP: `conditional-update-fhir-resource` with the search criteria + body.
   - REST: `PUT <base>/fhir/Patient?identifier=http://hospital.org/mrn|12345`
     with the full Patient body. Aidbox matches on the search:
     - 0 matches → creates (201)
     - 1 match → updates that resource (200)
     - >1 match → `412`/`OperationOutcome` (ambiguous — tighten the key)
3. **Guard concurrent writers** with `If-Match: W/"<versionId>"` for optimistic locking
   (`412` on version conflict — re-read and retry).

## Rules
- Conditional update + `If-None-Exist` are the idempotency mechanisms here
  (`conventions/aidbox-conventions.yml`); never POST the same record twice without a key.
- Always include the identifying `identifier` in the body so the created/updated resource
  carries the same business key you searched on.
