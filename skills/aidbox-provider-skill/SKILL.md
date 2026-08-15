---
name: aidbox
description: Use when working with Aidbox FHIR server - setup, data loading, access control, SQL queries, subscriptions, multi-tenancy, or troubleshooting. Knows Aidbox-specific patterns that differ from standard FHIR.
---

# Aidbox Development

Aidbox is a FHIR server on PostgreSQL with REST, SQL, and GraphQL APIs.

---

## 📋 Capabilities

| Category | Features |
|----------|----------|
| **FHIR** | STU3, R4, R4B, R5, R6 • CRUD, history, transactions • _include, _revinclude, _has, chained search • Structured Data Capture |
| **Data APIs** | REST API • SQL API • GraphQL API • SQL-on-FHIR (ViewDefinition) |
| **Bulk Operations** | $import (~21k/sec) • $export • $load • ndjson, gzip |
| **Terminology** | 500+ FHIR IGs • CodeSystem, ValueSet, ConceptMap • External terminology servers |
| **Customization** | Custom resources • SearchParameters • AidboxTrigger • Aidbox Apps |
| **Subscriptions** | Topic-based • Kafka, Webhook, Pub/Sub, AMQP, EventBridge • Changes API (polling) |
| **Security** | OAuth 2.0, OIDC, Basic Auth, SSO • RBAC, ABAC, label-based • AccessPolicy (Matcho, SQL) |
| **Multi-tenancy** | OrgBAC • Hierarchical organizations • Isolated FHIR APIs per tenant |
| **Integration** | HL7 v2 inbound • C-CDA converter • X12 • MCP Server for AI agents |
| **Storage** | PostgreSQL JSONB • Read replicas • S3, GCS, Azure Blob file storage |
| **Deployment** | Docker, Kubernetes • AWS, Azure, GCP, on-prem • OpenTelemetry |
| **Forms** | Aidbox Forms • UI Builder • AI assistance • Offline support |
| **SDKs** | TypeScript SDK • Python SDK |
| **Compliance** | HIPAA, HITECH, GDPR • ISO 27001 certified |

---

## 🎯 What Are You Building?

Before diving into code, identify your goal:

| Goal | Start Here |
|------|------------|
| **Run Aidbox locally** | → [getting-started.md](getting-started.md) |
| **Set up authentication** | → [getting-started.md](getting-started.md#authentication) |
| **Control who sees what data** | → [access-policy.md](access-policy.md) |
| **Multi-tenant application** | → [orgbac.md](orgbac.md) |
| **Load/export bulk data** | → [bulk-operations.md](bulk-operations.md) |
| **Direct SQL queries** | → [sql-api.md](sql-api.md) |
| **Real-time notifications** | → [subscriptions.md](subscriptions.md) |
| **Custom resources/profiles** | → [far-api.md](far-api.md) |
| **Connect AI agent to Aidbox** | → [mcp-setup.md](mcp-setup.md) |
| **Debug slow queries** | → [troubleshooting.md](troubleshooting.md) |
| **Configure environment** | → [configuration.md](configuration.md) |
| **Ready-to-use examples** | → [examples/](../../examples/) |

---

## ⚠️ Critical Rules

**DO NOT:**
- ❌ Use `engine: allow` in production (dev only, requires `BOX_SECURITY_DEV_MODE=true`)
- ❌ Create global AccessPolicies (always link to User/Client)
- ❌ Use unanchored regex in Matcho (`/fhir/` → `#^/fhir/.*$`)
- ❌ Skip `present?` checks in Matcho patterns
- ❌ Use `_total=accurate` on large datasets without need
- ❌ Use `/Patient/*` for external integrations (use `/fhir/Patient/*`)

**ALWAYS:**
- ✅ Test AccessPolicy with `/$matcho` before deploying
- ✅ Debug access issues with `?__debug=policy`
- ✅ Use `/fhir/*` endpoints for FHIR compliance
- ✅ Link AccessPolicies to specific User/Client
- ✅ Use `_explain=analyze` for slow queries

---

## 🚀 Common Workflows

### Start New Project

**Step 1: Download (if needed) and start**
```bash
test -f docker-compose.yml || curl -JO https://aidbox.app/runme
docker compose up -d
```

**Step 2: Wait for startup (healthcheck)**
```bash
until curl -sf http://localhost:8080/health > /dev/null 2>&1; do sleep 2; done
```
This polls healthcheck every 2 seconds until Aidbox is ready.

**Step 3: License activation required**
- Tell user to open http://localhost:8080
- Click "Sign in with AidboxID" and sign in at https://aidbox.app
- **ASK user to confirm** when activation is complete

**Step 4: Get root client secret**
```bash
export AIDBOX_SECRET=$(docker compose config | grep BOX_ROOT_CLIENT_SECRET | awk '{print $2}')
```
Use `-u root:$AIDBOX_SECRET` in all API calls.

See: [getting-started.md](getting-started.md)

### Add Access Control
1. Read [access-policy.md](access-policy.md)
2. Choose engine: **Matcho** (90% cases) or **SQL** (complex rules)
3. Write policy with `link` to User/Client
4. Test with `POST /$matcho`
5. Debug with `?__debug=policy`

### Load Test Data

**Step 1: Start import** - copy EXACTLY, do not modify or reorder:
```bash
curl -X POST 'http://localhost:8080/v2/fhir/$import' \
  -H 'Content-Type: application/json' \
  -u root:$AIDBOX_SECRET \
  -d '{"id":"synthea-import","contentEncoding":"gzip","inputs":[
    {"resourceType":"AllergyIntolerance","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/AllergyIntolerance.ndjson.gz"},
    {"resourceType":"CarePlan","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/CarePlan.ndjson.gz"},
    {"resourceType":"CareTeam","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/CareTeam.ndjson.gz"},
    {"resourceType":"Claim","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Claim.ndjson.gz"},
    {"resourceType":"Condition","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Condition.ndjson.gz"},
    {"resourceType":"Device","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Device.ndjson.gz"},
    {"resourceType":"DiagnosticReport","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/DiagnosticReport.ndjson.gz"},
    {"resourceType":"DocumentReference","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/DocumentReference.ndjson.gz"},
    {"resourceType":"Encounter","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Encounter.ndjson.gz"},
    {"resourceType":"ExplanationOfBenefit","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/ExplanationOfBenefit.ndjson.gz"},
    {"resourceType":"ImagingStudy","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/ImagingStudy.ndjson.gz"},
    {"resourceType":"Immunization","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Immunization.ndjson.gz"},
    {"resourceType":"Location","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Location.ndjson.gz"},
    {"resourceType":"Medication","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Medication.ndjson.gz"},
    {"resourceType":"MedicationAdministration","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/MedicationAdministration.ndjson.gz"},
    {"resourceType":"MedicationRequest","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/MedicationRequest.ndjson.gz"},
    {"resourceType":"Observation","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Observation.ndjson.gz"},
    {"resourceType":"Organization","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Organization.ndjson.gz"},
    {"resourceType":"Patient","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Patient.ndjson.gz"},
    {"resourceType":"Practitioner","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Practitioner.ndjson.gz"},
    {"resourceType":"PractitionerRole","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/PractitionerRole.ndjson.gz"},
    {"resourceType":"Procedure","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Procedure.ndjson.gz"},
    {"resourceType":"Provenance","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/Provenance.ndjson.gz"},
    {"resourceType":"SupplyDelivery","url":"https://storage.googleapis.com/aidbox-public/synthea/v2/100/fhir/SupplyDelivery.ndjson.gz"}
  ]}'
```

**Step 2: Wait for import and show results**

Copy this script EXACTLY - do not modify or add variables:
```bash
while true; do
  resp=$(curl -s 'http://localhost:8080/v2/$import/synthea-import' -u root:$AIDBOX_SECRET)
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

### Debug Slow Search
```bash
curl 'http://localhost:8080/fhir/Patient?name=John&_explain=analyze' -u root:$AIDBOX_SECRET
```
See: [troubleshooting.md](troubleshooting.md)

---

## 🔑 Key Aidbox Concepts

### Dual API Format
| Endpoint | Format | Use For |
|----------|--------|---------|
| `/fhir/Patient/123` | Standard FHIR | External integrations, compliance |
| `/Patient/123` | Aidbox native | Internal, extensions as fields |

**Rule**: Always use `/fhir/*` for external systems.

### Everything is a Resource
AccessPolicy, Client, SearchParameter, Settings — all manageable via REST API.

### Performance Reference
| Operation | Speed | Use Case |
|-----------|-------|----------|
| Transaction bundle | ~3,500/sec | Atomicity needed |
| $import | ~21,000/sec | Bulk load, no validation |
| $load | ~10,000/sec | Streaming sync |
| $export | ~15,500/sec | Bulk export to cloud |

---

## 📚 Detailed Guides

| Guide | Content |
|-------|---------|
| [getting-started.md](getting-started.md) | Docker, runme, auth setup |
| [configuration.md](configuration.md) | Environment variables, Init Bundle |
| [access-policy.md](access-policy.md) | Matcho, SQL engine, patterns |
| [orgbac.md](orgbac.md) | Multi-tenancy, organization scopes |
| [sql-api.md](sql-api.md) | $sql, AidboxQuery, SQL on FHIR |
| [bulk-operations.md](bulk-operations.md) | $import, $export, Synthea data |
| [subscriptions.md](subscriptions.md) | Kafka, Webhook, Pub/Sub |
| [far-api.md](far-api.md) | FHIR packages, custom resources |
| [mcp-setup.md](mcp-setup.md) | AI agent integration |
| [troubleshooting.md](troubleshooting.md) | Performance, indexing, debugging |

---

## 📦 Ready-to-Use Examples

Copy-paste examples in [examples/](../../examples/):

| File | Description |
|------|-------------|
| `init-bundle.json` | Batch bundle with Client + AccessPolicy |
| `access-policies/patient-own-data.json` | Patient views own data |
| `access-policies/practitioner-own-patients.json` | Practitioner sees own patients |
| `access-policies/practitioner-all-observations.json` | Practitioner views all observations |
| `access-policies/client-crud-resources.json` | Client CRUD on Patient/Practitioner |
| `access-policies/jwt-issuer-based.json` | JWT issuer validation |
| `access-policies/sql-read-only.json` | SQL endpoint read-only |
