---
name: Create and read a FHIR Patient
description: Authenticate to an Aidbox box and create then read back a FHIR Patient resource, via MCP tools or the FHIR REST API.
api: fhir/aidbox-capabilitystatement.json
operations: [create-fhir-resource, read-fhir-resource]
mcp_tools: [create-fhir-resource, read-fhir-resource]
---

# Create and read a FHIR Patient (Aidbox)

Use this to write a patient record into an Aidbox FHIR box and confirm it persisted.

## Auth
Obtain an OAuth2 / SMART bearer token from the box's token endpoint
(`<base>/auth/token`). Backend jobs use `client_credentials` with `private_key_jwt`;
interactive apps use SMART App Launch. Send `Authorization: Bearer <token>`.
See `authentication/aidbox-authentication.yml` and `scopes/aidbox-scopes.yml` — writing
Patient needs a scope covering `Patient` create (e.g. `system/Patient.c` or `user/Patient.c`).

## Steps
1. **Create** the Patient.
   - MCP: call `create-fhir-resource` with `resourceType: Patient` and the body.
   - REST: `POST <base>/fhir/Patient` with `Content-Type: application/fhir+json`:
     ```json
     {"resourceType":"Patient","name":[{"family":"Doe","given":["Jane"]}],"gender":"female"}
     ```
   - The 201 response carries the server-assigned `id` and `meta.versionId` (capture both).
2. **Read** it back.
   - MCP: call `read-fhir-resource` with `resourceType: Patient`, `id: <id>`.
   - REST: `GET <base>/fhir/Patient/{id}` → the stored resource with its `ETag`.

## Rules
- Errors come back as a FHIR `OperationOutcome` (see `errors/aidbox-problem-types.yml`);
  `422` means the resource failed validation — inspect `issue[].diagnostics`.
- Do not hardcode a client id in the body; the server assigns `id` on create.
