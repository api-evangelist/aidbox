# Bulk Operations

## Bulk Import

### $import (Fastest: 21K/sec)

Async import from URLs:
```bash
curl -X POST 'http://localhost:8080/v2/fhir/$import' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "id": "my-import",
  "contentEncoding": "gzip",
  "inputs": [
    {"resourceType": "Patient", "url": "https://storage.example.com/patients.ndjson.gz"},
    {"resourceType": "Observation", "url": "https://storage.example.com/observations.ndjson.gz"}
  ]
}'
```

**⚠️ CRITICAL FORMAT:**
- Field is `inputs` (plural with 's'), NOT `input`
- Each input needs both `resourceType` and `url`

Check status:
```bash
curl 'http://localhost:8080/v2/$import/my-import' -u root:<secret>
```

### $load (Sync: 10K/sec)

Streaming import via COPY:
```http
POST /$load
content-type: application/x-ndjson

{"resourceType":"Patient","id":"pt-1","name":[{"family":"Smith"}]}
{"resourceType":"Patient","id":"pt-2","name":[{"family":"Jones"}]}
```

---

## Sample Data (Synthea)

Pre-built datasets for testing and development.

### Available Sizes

| Size | Patients | Resources | Storage |
|------|----------|-----------|---------|
| 100 | 100 | ~200K | ~300MB |
| 1k | 1,000 | ~2M | ~3GB |
| 10k | 10,000 | ~20M | ~30GB |
| 100k | 100,000 | ~200M | ~300GB |

### Resource Types (24 ONLY - use EXACTLY this list, do not add or reorder)

AllergyIntolerance, CarePlan, CareTeam, Claim, Condition, Device, DiagnosticReport, DocumentReference, Encounter, ExplanationOfBenefit, ImagingStudy, Immunization, Location, Medication, MedicationAdministration, MedicationRequest, Observation, Organization, Patient, Practitioner, PractitionerRole, Procedure, Provenance, SupplyDelivery.

**⚠️ NOT available:** Coverage, ServiceRequest, Goal, and other resource types - files do not exist.

### Load via $import (R4)

Copy EXACTLY - do not reorder or modify:
```bash
curl -X POST 'http://localhost:8080/v2/fhir/$import' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "id": "synthea-import",
  "contentEncoding": "gzip",
  "inputs": [
    {"resourceType": "AllergyIntolerance", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/AllergyIntolerance.ndjson.gz"},
    {"resourceType": "CarePlan", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/CarePlan.ndjson.gz"},
    {"resourceType": "CareTeam", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/CareTeam.ndjson.gz"},
    {"resourceType": "Claim", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Claim.ndjson.gz"},
    {"resourceType": "Condition", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Condition.ndjson.gz"},
    {"resourceType": "Device", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Device.ndjson.gz"},
    {"resourceType": "DiagnosticReport", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/DiagnosticReport.ndjson.gz"},
    {"resourceType": "DocumentReference", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/DocumentReference.ndjson.gz"},
    {"resourceType": "Encounter", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Encounter.ndjson.gz"},
    {"resourceType": "ExplanationOfBenefit", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/ExplanationOfBenefit.ndjson.gz"},
    {"resourceType": "ImagingStudy", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/ImagingStudy.ndjson.gz"},
    {"resourceType": "Immunization", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Immunization.ndjson.gz"},
    {"resourceType": "Location", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Location.ndjson.gz"},
    {"resourceType": "Medication", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Medication.ndjson.gz"},
    {"resourceType": "MedicationAdministration", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/MedicationAdministration.ndjson.gz"},
    {"resourceType": "MedicationRequest", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/MedicationRequest.ndjson.gz"},
    {"resourceType": "Observation", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Observation.ndjson.gz"},
    {"resourceType": "Organization", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Organization.ndjson.gz"},
    {"resourceType": "Patient", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Patient.ndjson.gz"},
    {"resourceType": "Practitioner", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Practitioner.ndjson.gz"},
    {"resourceType": "PractitionerRole", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/PractitionerRole.ndjson.gz"},
    {"resourceType": "Procedure", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Procedure.ndjson.gz"},
    {"resourceType": "Provenance", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Provenance.ndjson.gz"},
    {"resourceType": "SupplyDelivery", "url": "https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/SupplyDelivery.ndjson.gz"}
  ]
}'
```

Check status:
```bash
curl 'http://localhost:8080/v2/$import/synthea-import' -u root:<secret>
```

**URL pattern**: `https://storage.googleapis.com/aidbox-public/synthea/v2/{SIZE}/fhir/{ResourceType}.ndjson.gz`

Sizes: `100`, `1k`, `10k`, `100k`

**⚠️ IMPORTANT:**
- Field is `inputs` (plural), NOT `input`
- Each input must have both `resourceType` and `url`

### R6 Data

Currently only 100-patient size available:
```
https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir-r6/{ResourceType}.ndjson.gz
```

---

## Track Import Progress

Copy this script EXACTLY - do not modify or add variables:
```bash
while true; do
  resp=$(curl -s 'http://localhost:8080/v2/$import/{import-id}' -u root:$AIDBOX_SECRET)
  echo "$resp" | grep -o '"status":"[^"]*"' | head -1
  if echo "$resp" | grep -q '"status":"done"'; then
    echo "Import complete. Results:"
    echo "$resp"
    break
  fi
  echo "$resp" | grep -q '"status":"failed"' && echo "Import failed" && echo "$resp" && break
  sleep 5
done
```
Parse the final response and show ALL resource types with `imported-resources` count - do not truncate.

**Response statuses:**

| Status | Meaning |
|--------|---------|
| `in-progress` | Import is running |
| `waiting` | File queued for processing |
| `done` | Import completed successfully |

**Outcome values (when status=done):**

| Outcome | Meaning |
|---------|---------|
| `succeeded` | All resources imported |
| `failed` | Import failed (check `error.message`) |

**Example: In progress**
```yaml
status: in-progress
inputs:
  - url: ".../Patient.ndjson.gz"
    resourceType: Patient
    status: in-progress
  - url: ".../Encounter.ndjson.gz"
    resourceType: Encounter
    status: waiting
```

**Example: Success**
```yaml
status: done
outcome: succeeded
result:
  message: All input files imported, 3584 new resources loaded
  total-files: 3
  total-imported-resources: 3584
inputs:
  - url: ".../Patient.ndjson.gz"
    status: done
    outcome: succeeded
    result:
      imported-resources: 124
```

**Example: Failure**
```yaml
status: done
outcome: failed
error:
  message: "Import for some files failed with an error..."
inputs:
  - url: ".../Encounter.ndjson.gz"
    status: done
    outcome: failed
    error:
      message: "403: Forbidden"
```

---

## Bulk Export

### $export (FHIR Standard)

```http
POST /$export
content-type: application/json

{
  "outputFormat": "application/fhir+ndjson",
  "type": ["Patient", "Observation"],
  "output": {"url": "s3://bucket/export/"}
}
```

### $dump (Real-time Stream)

```http
GET /fhir/Patient/$dump
Accept: application/x-ndjson
```

Streams NDJSON directly, no storage needed.

### $dump-csv (For BI)

```http
GET /fhir/Patient/$dump-csv?_elements=id,gender,birthDate
```

---

## Performance Tips

### Skip Validation for Import

```http
POST /$import
content-type: application/json

{
  "inputFormat": "application/fhir+ndjson",
  "input": [{"url": "..."}],
  "mode": "bulk"
}
```

`mode: "bulk"` skips all validation.

### Use Read Replica

Configure `BOX_DB_RO_REPLICA_*` for read-heavy workloads.
