# SQL API

## Basic Query
```bash
curl -X POST 'http://localhost:8080/$sql' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '["SELECT id, resource->>'"'"'gender'"'"' as gender FROM patient LIMIT 10"]'
```

## Parameterized Query
```bash
curl -X POST 'http://localhost:8080/$sql' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '["SELECT * FROM patient WHERE resource->>'"'"'gender'"'"' = ?", "male"]'
```

## JSON Operations

| Operator | Description | Example |
|----------|-------------|---------|
| `->` | Get JSON object | `resource->'name'` |
| `->>` | Get as text | `resource->>'gender'` |
| `#>` | Get at path | `resource#>'{name,0,given}'` |
| `#>>` | Get at path as text | `resource#>>'{name,0,family}'` |
| `@>` | Contains | `resource @> '{"active":true}'` |
| `?` | Key exists | `resource ? 'deceased'` |

## Table Structure

Each resource type has two tables:
- `patient` - Current versions
- `patient_history` - All versions

Columns: `id`, `txid`, `ts`, `resource`, `status`

## Knife Functions

```sql
SELECT knife_extract_text(resource, '[["name", "family"]]') as family
FROM patient
```

---

## AidboxQuery (Custom Search)

Define reusable SQL-based searches:

```bash
curl -X PUT 'http://localhost:8080/AidboxQuery/patient-by-org' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxQuery",
  "id": "patient-by-org",
  "params": {"org": {"type": "string", "isRequired": true}},
  "query": "SELECT * FROM patient WHERE resource#>>'"'"'{managingOrganization,id}'"'"' = {{params.org}}"
}'
```

Execute:
```bash
curl 'http://localhost:8080/$query/patient-by-org?org=org-123' -u root:<secret>
```

---

## SQL on FHIR (Analytics)

SQL on FHIR allows creating flat views from FHIR resources for analytics, reporting, and BI integration.

### ViewDefinition

```bash
curl -X POST 'http://localhost:8080/ViewDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "ViewDefinition",
  "name": "patient_demographics",
  "status": "draft",
  "resource": "Patient",
  "select": [
    {"column": [
      {"name": "id", "path": "getResourceKey()"},
      {"name": "gender", "path": "gender"},
      {"name": "birth_date", "path": "birthDate"}
    ]},
    {"forEach": "name.where(use = '"'"'official'"'"').first()",
     "column": [
       {"name": "family_name", "path": "family"},
       {"name": "given_name", "path": "given.join('"'"' '"'"')"}
     ]}
  ]
}'
```

### $run Operation (Query without materialization)

```bash
curl -X POST 'http://localhost:8080/fhir/ViewDefinition/patient_demographics/$run' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"Parameters","parameter":[{"name":"_format","valueCode":"json"},{"name":"_limit","valueInteger":100}]}'
```

**Parameters**:
- `_format`: `json` | `ndjson` | `csv`
- `_limit`: max rows
- `_since`: only resources updated after date
- `patient`: filter by patient compartment
- `group`: filter by Group membership

### $materialize Operation (Create database objects)

```bash
curl -X POST 'http://localhost:8080/fhir/ViewDefinition/patient_demographics/$materialize' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"Parameters","parameter":[{"name":"type","valueCode":"view"}]}'
```

**Types**:
- `view` - PostgreSQL view (always up-to-date)
- `materialized-view` - Cached, needs refresh
- `table` - Static table (fastest queries)

**Default schema**: `sof` (configurable via `BOX_VIEW_DEFINITION_SCHEMA`)

### Query Materialized Views

```sql
SELECT * FROM sof.patient_demographics
WHERE gender = 'female'
ORDER BY birth_date DESC;
```

### FHIRPath in ViewDefinition

Common FHIRPath expressions:
- `getResourceKey()` - Resource ID
- `gender` - Simple field
- `birthDate` - Date field
- `name.first().family` - First name family
- `name.where(use = 'official').first().given.join(' ')` - Official given names
- `identifier.where(system = 'http://mrn').value` - MRN identifier
- `address.where(use = 'home').first().city` - Home city

### ViewDefinition Builder

Online tool: https://sqlonfhir.aidbox.app/

Features:
- Visual builder
- FHIRPath autocomplete
- Live preview
- Export to Aidbox
