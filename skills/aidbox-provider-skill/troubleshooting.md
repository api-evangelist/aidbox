# Troubleshooting Aidbox

## Performance Issues

### Slow Search Queries

**Diagnose with _explain:**
```bash
curl 'http://localhost:8080/fhir/Patient?name=John&_explain=analyze' -u root:<secret>
```

Returns SQL query plan with execution times.

### Common Index Patterns

#### GIN Index for JSONB Search
```sql
-- For name search
CREATE INDEX patient_name_idx ON patient
USING gin ((resource->'name') jsonb_path_ops);

-- For identifier search
CREATE INDEX patient_identifier_idx ON patient
USING gin ((resource->'identifier') jsonb_path_ops);

-- For code search (Observation, Condition, etc.)
CREATE INDEX observation_code_idx ON observation
USING gin ((resource->'code') jsonb_path_ops);
```

#### B-tree Index for Date/Exact Match
```sql
-- For birthDate
CREATE INDEX patient_birthdate_idx ON patient
((resource->>'birthDate'));

-- For status
CREATE INDEX observation_status_idx ON observation
((resource->>'status'));

-- For lastUpdated
CREATE INDEX patient_lastupdated_idx ON patient (ts);
```

#### Expression Index for Nested Paths
```sql
-- For gender
CREATE INDEX patient_gender_idx ON patient
((resource->>'gender'));

-- For reference
CREATE INDEX observation_subject_idx ON observation
((resource#>>'{subject,reference}'));
```

#### Custom Search Resource (Alternative to Index)
```bash
curl -X PUT 'http://localhost:8080/Search/Patient.name' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"Search","id":"Patient.name","resource":{"resourceType":"Entity","id":"Patient"},"expression":[["name"]],"index-type":"gin"}'
```

### Timeout Issues

#### $everything Timeout

**Problem:** `GET /fhir/Patient/pt-1/$everything` times out

**Solutions:**

1. Use pagination:
```bash
curl 'http://localhost:8080/fhir/Patient/pt-1/$everything?_count=100' -u root:<secret>
```

2. Filter by type:
```bash
curl 'http://localhost:8080/fhir/Patient/pt-1/$everything?_type=Observation,Condition' -u root:<secret>
```

3. Filter by date:
```bash
curl 'http://localhost:8080/fhir/Patient/pt-1/$everything?start=2024-01-01' -u root:<secret>
```

#### _revinclude Timeout

**Problem:** `GET /fhir/Patient?_revinclude=Observation:subject` times out

**Solutions:**

1. Add index on reference:
```sql
CREATE INDEX observation_subject_idx ON observation
USING gin ((resource->'subject') jsonb_path_ops);
```

2. Use smaller page size:
```bash
curl 'http://localhost:8080/fhir/Patient?_revinclude=Observation:subject&_count=10' -u root:<secret>
```

3. Use Custom Search with prebuilt index:
```bash
curl -X PUT 'http://localhost:8080/Search/Observation.subject' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"Search","resource":{"resourceType":"Entity","id":"Observation"},"expression":[["subject","id"]],"index-type":"btree"}'
```

#### $lookup Slow on Organization

**Problem:** `GET /fhir/Organization/$lookup` is slow

**Solution:** Create index:
```sql
CREATE INDEX organization_name_idx ON organization
USING gin ((resource->'name') gin_trgm_ops);
```

### _count Performance

**Problem:** Large `_count` values slow down queries

**Solutions:**

1. Use `_total=none` to skip count query:
```bash
curl 'http://localhost:8080/fhir/Patient?name=John&_total=none' -u root:<secret>
```

2. Use `_total=estimate` for approximate count:
```bash
curl 'http://localhost:8080/fhir/Patient?name=John&_total=estimate' -u root:<secret>
```

3. Set default in environment:
```yaml
BOX_FHIR_SEARCH_DEFAULT_PARAMS_TOTAL: 'estimate'
BOX_FHIR_SEARCH_DEFAULT_PARAMS_COUNT: 50
```

---

## AccessPolicy Issues

### Debug AccessPolicy

```bash
curl 'http://localhost:8080/fhir/Patient/123?__debug=policy' -u root:<secret>
# or with header:
curl 'http://localhost:8080/fhir/Patient/123' -H 'x-debug: policy' -u root:<secret>
```

### Test Matcho Pattern

```bash
curl -X POST 'http://localhost:8080/$matcho' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resource": {
    "request-method": "get",
    "uri": "/fhir/Patient/pt-123",
    "user": {"data": {"patient_id": "pt-123"}}
  },
  "matcho": {
    "uri": "#^/fhir/Patient/.user.data.patient_id$"
  }
}'
```

### Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Unanchored regex | `/Patient/` matches `/fhir/Patient/123/history` | `#^/fhir/Patient/[^/]+$` |
| Missing `present?` | Fails on null values | `{present?: true, ...}` |
| Global policy | Lower priority, runs for all requests | Link to User/Client |
| Wrong context path | `.user.patient_id` doesn't exist | `.user.data.patient_id` |

### Policy Not Linked

**Problem:** Policy runs for every request (slow)

**Fix:** Link to User/Client:
```bash
curl -X PUT 'http://localhost:8080/AccessPolicy/patient-access' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"AccessPolicy","id":"patient-access","engine":"matcho","matcho":{"uri":"#^/fhir/Patient/.*$"},"link":[{"resourceType":"User","id":"patient-user-1"}]}'
```

---

## Validation Issues

### Skip Reference Validation

```bash
curl -X POST 'http://localhost:8080/fhir/Patient' \
  -H 'Content-Type: application/json' \
  -H 'Skip-Validation: reference' \
  -u root:<secret> \
  -d '{"resourceType":"Patient","managingOrganization":{"reference":"Organization/not-exists"}}'
```

### Skip All Validation (Bulk Import)

```bash
curl -X POST 'http://localhost:8080/v2/fhir/$import' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"id":"bulk-import","contentEncoding":"gzip","inputs":[{"resourceType":"Patient","url":"..."}]}'
```

### Extension Validation Errors

**Problem:** "Unknown extension" error

**Solutions:**

1. Load the StructureDefinition for the extension:
```bash
curl -X POST 'http://localhost:8080/fhir/StructureDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "StructureDefinition",
  "url": "http://example.org/StructureDefinition/my-extension",
  "name": "MyExtension",
  "status": "active",
  "kind": "complex-type",
  "abstract": false,
  "context": [{"type": "element", "expression": "Patient"}],
  "type": "Extension",
  "baseDefinition": "http://hl7.org/fhir/StructureDefinition/Extension",
  "derivation": "constraint",
  "differential": {
    "element": [
      {"id": "Extension.value[x]", "path": "Extension.value[x]", "type": [{"code": "string"}]}
    ]
  }
}'
```

2. Or disable extension validation (not recommended):
```yaml
BOX_FHIR_SCHEMA_VALIDATION: 'false'
```

### FHIRSchema Null Values

**Problem:** `null` values cause validation errors in FHIRSchema mode (2405+)

**Solution:** Strip nulls before sending:
```javascript
// JavaScript
const cleanResource = JSON.parse(JSON.stringify(resource, (k, v) => v === null ? undefined : v));
```

---

## Format Issues

### Aidbox vs FHIR Format

**Problem:** Reference format mismatch

| Aspect | FHIR (`/fhir/*`) | Aidbox (`/*`) |
|--------|------------------|---------------|
| Reference | `{"reference": "Patient/123"}` | `{"id": "123", "resourceType": "Patient"}` |

**Solution:** Use `/fhir/*` endpoints for FHIR-compliant format.

**Enable auto-correction:**
```yaml
BOX_FHIR_CORRECT_AIDBOX_FORMAT: 'true'
```

---

## Bundle/Transaction Issues

### Transaction Failed

**Debug:**
```bash
curl -X POST 'http://localhost:8080/fhir' \
  -H 'Content-Type: application/json' \
  -H 'Prefer: return=OperationOutcome' \
  -u root:<secret> \
  -d '{"resourceType":"Bundle","type":"transaction","entry":[...]}'
```

**Solutions:**

1. Use `batch` instead of `transaction` to continue on error
2. Validate resources individually first
3. Check for circular references

### Bundle Too Large

**Problem:** Large bundle causes timeout or memory issues

**Solutions:**

1. Split into smaller batches (1000 resources per batch)
2. Use `$import` for bulk operations
3. Use `$load` for streaming:
```bash
curl -X POST 'http://localhost:8080/$load' \
  -H 'Content-Type: application/x-ndjson' \
  -u root:<secret> \
  -d '{"resourceType":"Patient","id":"pt-1"}
{"resourceType":"Patient","id":"pt-2"}'
```

---

## SearchParameter Issues

### SearchParameter Not Found

**Check:**
```bash
curl 'http://localhost:8080/SearchParameter?code=<param-name>&base=<resource-type>' -u root:<secret>
```

**Create:**
```bash
curl -X PUT 'http://localhost:8080/fhir/SearchParameter/patient-identifier' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "SearchParameter",
  "url": "http://hl7.org/fhir/SearchParameter/Patient-identifier",
  "name": "identifier",
  "status": "active",
  "base": ["Patient"],
  "type": "token",
  "expression": "Patient.identifier"
}'
```

### Search by Extension Not Working

**Problem:** Can't search by extension value

**Solution:** Create SearchParameter for extension:
```bash
curl -X PUT 'http://localhost:8080/fhir/SearchParameter/patient-occupation' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "SearchParameter",
  "url": "http://example.org/SearchParameter/patient-occupation",
  "name": "occupation",
  "status": "active",
  "code": "occupation",
  "base": ["Patient"],
  "type": "string",
  "expression": "Patient.extension.where(url='"'"'http://example.org/occupation'"'"').value.as(string)"
}'
```

---

## Export Issues

### Memory Issues with Large Exports

**Solutions:**

1. Stream to cloud storage:
```bash
curl -X POST 'http://localhost:8080/$export' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"output":{"url":"s3://bucket/export/"},"type":["Patient"]}'
```

2. Use $dump for real-time streaming:
```bash
curl 'http://localhost:8080/fhir/Patient/$dump' \
  -H 'Accept: application/x-ndjson' \
  -u root:<secret>
```

3. Use ViewDefinition + materialize for analytics

---

## Performance Checklist

- [ ] Indexes on frequently searched fields
- [ ] AccessPolicies linked to User/Client (not global)
- [ ] Read replica for read-heavy workloads (`BOX_DB_RO_REPLICA_*`)
- [ ] `$import` instead of transactions for bulk
- [ ] ViewDefinition materialized for analytics
- [ ] `_total=none` or `_total=estimate` for searches
- [ ] Connection pool sized correctly (`BOX_DB_POOL_MAXIMUM_POOL_SIZE`)
- [ ] Adequate PostgreSQL resources

---

## Diagnostic Queries

### Check Table Sizes
```sql
SELECT relname, pg_size_pretty(pg_total_relation_size(relid))
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(relid) DESC;
```

### Check Slow Queries
```sql
SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

### Check Index Usage
```sql
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

### Check Missing Indexes
```sql
SELECT relname, seq_scan, seq_tup_read, idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > 0
ORDER BY seq_tup_read DESC;
```

### Check Resource Counts
```bash
curl 'http://localhost:8080/fhir/Patient?_count=0&_total=accurate' -u root:<secret>
# Returns total in Bundle.total
```
