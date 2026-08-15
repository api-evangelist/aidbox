# AccessPolicy Reference

Aidbox uses AccessPolicy resources for authorization. Default: **deny all**.

## Iron Rules

1. **NEVER use `engine: allow` in production** - only for development
2. **ALWAYS link policies to User/Client** - unlinked = global = lower priority = slower
3. **ALWAYS test with `/$matcho`** before deploying
4. **Use `__debug=policy`** to troubleshoot
5. **Anchor regex patterns** - use `^` and `$` to match whole string

---

## Engines

### 1. Allow (Development Only!)
```json
{
  "resourceType": "AccessPolicy",
  "id": "allow-all",
  "engine": "allow"
}
```
Requires `BOX_SECURITY_DEV_MODE=true` to work.

### 2. Matcho (90% of Use Cases)

Pattern matching DSL against request object.

```json
{
  "resourceType": "AccessPolicy",
  "id": "patient-read-own",
  "engine": "matcho",
  "matcho": {
    "request-method": "get",
    "uri": "#^/fhir/Patient/.user.data.patient_id$",
    "user": {"data": {"patient_id": {"present?": true}}}
  },
  "link": [{"resourceType": "User", "id": "patient-user"}]
}
```

### 3. SQL (Complex Rules)

```json
{
  "resourceType": "AccessPolicy",
  "id": "practitioner-careteam",
  "engine": "sql",
  "sql": {
    "query": "SELECT EXISTS (SELECT 1 FROM careteam ct WHERE ct.resource#>>'{participant,0,member,id}' = {{user.id}} AND ct.resource#>>'{subject,id}' = {{params.patient}})"
  },
  "link": [{"resourceType": "User", "id": "practitioner-user"}]
}
```

### 4. Complex (Combine Engines)

```json
{
  "resourceType": "AccessPolicy",
  "id": "complex-policy",
  "engine": "complex",
  "and": [
    {"engine": "matcho", "matcho": {"request-method": "get"}},
    {"engine": "sql", "sql": {"query": "SELECT {{user.data.active}}"}}
  ]
}
```

### 5. JSON Schema

```json
{
  "resourceType": "AccessPolicy",
  "id": "schema-policy",
  "engine": "json-schema",
  "schema": {
    "properties": {
      "request-method": {"enum": ["get"]},
      "uri": {"pattern": "^/fhir/Patient.*$"}
    }
  }
}
```

---

## Request Object Structure

```yaml
# Full request object available in AccessPolicy
request-method: get | post | put | patch | delete
uri: "/fhir/Patient/pt-123"
params: {_count: "10", name: "John"}
headers: {authorization: "Bearer ...", x-custom: "value"}
body: {resourceType: "Patient", ...}

# Parsed from request:
resource: {resourceType: "Patient", id: "pt-123", ...}  # Target resource

# Authentication context:
user: {id: "user-1", data: {patient_id: "pt-123", ...}}
client: {id: "client-1", grant_types: [...]}
session: {id: "sess-1", ...}

# JWT claims:
jwt: {iss: "...", sub: "...", aud: "...", scope: "..."}
```

---

## Matcho Syntax

### Operators
| Operator | Description | Example |
|----------|-------------|---------|
| `#pattern` | Regex match | `"#^/fhir/Patient.*$"` |
| `.path` | Context reference | `".user.data.patient_id"` |
| `present?` | Value exists and not null | `{"patient_id": {"present?": true}}` |
| `nil?` | Value is null or missing | `{"deleted": {"nil?": true}}` |
| `$one-of` | Match any in list | `{"method": {"$one-of": ["get", "post"]}}` |
| `$contains` | String/array contains | `{"scope": {"$contains": "patient"}}` |
| `$not` | Negation | `{"$not": {"method": "delete"}}` |
| `$every` | All array elements match | `{"roles": {"$every": {"active": true}}}` |

### Interpolation in Regex
```yaml
# .path interpolates value from context
uri: "#^/fhir/Patient/.user.data.patient_id$"
#                     ↑ interpolates user.data.patient_id

# Example: if user.data.patient_id = "pt-123"
# Pattern becomes: "^/fhir/Patient/pt-123$"
```

---

## Common Patterns

### Patient Sees Only Own Data
```json
{
  "resourceType": "AccessPolicy",
  "id": "patient-own-data",
  "engine": "matcho",
  "matcho": {
    "request-method": "get",
    "uri": "#^/fhir/Patient/.user.data.patient_id$",
    "user": {"data": {"patient_id": {"present?": true}}}
  },
  "link": [{"resourceType": "User", "id": "patient-user"}]
}
```

### Practitioner Sees Patients via CareTeam
```json
{
  "resourceType": "AccessPolicy",
  "id": "practitioner-careteam",
  "engine": "sql",
  "sql": {
    "query": "SELECT EXISTS (SELECT 1 FROM careteam ct WHERE ct.resource#>>'{participant,0,member,id}' = {{user.id}} AND {{params}}->'patient' IS NOT NULL AND ct.resource#>>'{subject,id}' = ANY (SELECT jsonb_array_elements_text({{params}}->'patient')))"
  },
  "link": [{"resourceType": "User", "id": "practitioner-user"}]
}
```

### Organization-Based Access
```json
{
  "resourceType": "AccessPolicy",
  "id": "org-access",
  "engine": "sql",
  "sql": {
    "query": "SELECT ({{resource}}#>>'{managingOrganization,id}' = {{user.data.org_id}} OR {{resource}}#>>'{managingOrganization,id}' IS NULL)"
  },
  "link": [{"resourceType": "User", "id": "org-user"}]
}
```

### JWT Claim Validation
```json
{
  "resourceType": "AccessPolicy",
  "id": "jwt-validation",
  "engine": "matcho",
  "matcho": {
    "jwt": {
      "iss": "https://auth.example.com",
      "aud": "my-aidbox",
      "scope": {"$contains": "patient/*.read"}
    }
  }
}
```

### GraphQL Access
```json
{
  "resourceType": "AccessPolicy",
  "id": "graphql-policy",
  "engine": "matcho",
  "matcho": {
    "request-method": "post",
    "uri": "/$graphql",
    "body": {
      "query": {"$not": {"$contains": "Mutation"}}
    }
  }
}
```

### Consent-Based Access
```json
{
  "resourceType": "AccessPolicy",
  "id": "consent-based",
  "engine": "sql",
  "sql": {
    "query": "SELECT EXISTS (SELECT 1 FROM consent c WHERE c.resource#>>'{patient,id}' = {{params}}->'patient'->>0 AND c.resource#>>'{provision,actor,0,reference,id}' = {{user.id}} AND c.resource->>'status' = 'active')"
  }
}
```

### Allow All Reads, Restrict Writes
```json
{
  "resourceType": "AccessPolicy",
  "id": "read-only",
  "engine": "matcho",
  "matcho": {
    "request-method": {"$one-of": ["get", "head", "options"]},
    "uri": "#^/fhir/.*$"
  }
}
```

---

## Performance: Link Policies

Link policies to User/Client for faster evaluation:

```json
{
  "resourceType": "AccessPolicy",
  "id": "app-policy",
  "engine": "matcho",
  "matcho": {"uri": "#^/fhir/.*$"},
  "link": [
    {"resourceType": "Client", "id": "my-app"}
  ]
}
```

**Linked policies** are evaluated only for that User/Client.
**Global policies** (without link) are evaluated for EVERY request - slower!

---

## Debugging

### Enable Debug Output
```bash
curl 'http://localhost:8080/fhir/Patient/123?__debug=policy' -u root:<secret>
# or with header:
curl 'http://localhost:8080/fhir/Patient/123' -H 'x-debug: policy' -u root:<secret>
```

### Test Matcho Pattern Before Deploy
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

Response: `{"match": true}` or `{"match": false, "reason": "..."}`

---

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `uri: "#/fhir/Patient/"` | Not anchored, matches anywhere | `uri: "#^/fhir/Patient/.*$"` |
| `.user.patient_id` | Wrong path | `.user.data.patient_id` |
| No `present?` check | Fails on missing field | Add `user: {data: {patient_id: {present?: true}}}` |
| Global policy | Lower priority, slower | Link to User/Client |
| `$not` at wrong level | Negation scope wrong | Check nesting carefully |

---

## SMART on FHIR

Enable SMART authorization via environment variable:

```bash
BOX_SECURITY_SMART_ENABLED=true
```

Then use standard SMART authorization flows with scopes like `patient/*.read`.
