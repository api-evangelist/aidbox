# Organization-Based Access Control (OrgBAC)

Multi-tenancy with hierarchical organization-based access control to isolate FHIR data per tenant.

## Enable OrgBAC

```yaml
# Environment variable (FHIRSchema mode, Aidbox 2412+)
BOX_SECURITY_ORGBAC_ENABLED: 'true'
```

---

## How It Works

Each Organization gets a dedicated FHIR API:

```
# Organization-scoped FHIR API
<AIDBOX_BASE_URL>/Organization/<org-id>/fhir

# Organization-scoped Aidbox API
<AIDBOX_BASE_URL>/Organization/<org-id>/aidbox
```

### Hierarchy Example

```
org-a (parent)
├── org-b (child)
│   └── org-c (grandchild)
└── org-d (child)
```

- `org-a` API can see resources in org-a, org-b, org-c, org-d
- `org-b` API can see resources in org-b, org-c only
- `org-c` API can see resources in org-c only

---

## Create Organization Hierarchy

```bash
curl -X POST 'http://localhost:8080/fhir' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Bundle",
  "type": "batch",
  "entry": [
    {
      "request": {"method": "PUT", "url": "Organization/org-a"},
      "resource": {"resourceType": "Organization", "name": "Organization A"}
    },
    {
      "request": {"method": "PUT", "url": "Organization/org-b"},
      "resource": {
        "resourceType": "Organization",
        "name": "Organization B",
        "partOf": {"reference": "Organization/org-a"}
      }
    },
    {
      "request": {"method": "PUT", "url": "Organization/org-c"},
      "resource": {
        "resourceType": "Organization",
        "name": "Organization C",
        "partOf": {"reference": "Organization/org-b"}
      }
    }
  ]
}'
```

---

## CRUD Operations

### Create Resource in Organization

```bash
curl -X PUT 'http://localhost:8080/Organization/org-b/fhir/Patient/pt-1' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Patient",
  "name": [{"given": ["John"], "family": "Smith"}],
  "gender": "male"
}'
```

Resource gets `https://aidbox.app/tenant-organization-id` extension automatically:

```json
{
  "meta": {
    "extension": [
      {
        "url": "https://aidbox.app/tenant-organization-id",
        "valueReference": {"reference": "Organization/org-b"}
      }
    ]
  }
}
```

### Read from Parent (Works)

```bash
curl 'http://localhost:8080/Organization/org-a/fhir/Patient/pt-1' -u root:<secret>
# 200 OK - parent can see child's data
```

### Read from Sibling (Forbidden)

```bash
curl 'http://localhost:8080/Organization/org-d/fhir/Patient/pt-1' -u root:<secret>
# 403 Forbidden - siblings can't see each other's data
```

### Search

```bash
curl 'http://localhost:8080/Organization/org-a/fhir/Patient?gender=male' -u root:<secret>
# Returns patients from org-a, org-b, org-c, org-d
```

### Conditional Operations (2509+)

```bash
# Conditional create
curl -X POST 'http://localhost:8080/Organization/org-a/fhir/Observation' \
  -H 'Content-Type: application/json' \
  -H 'If-None-Exist: identifier=http://acme.org/obs|12345' \
  -u root:<secret> \
  -d '{"resourceType": "Observation", "status": "final"}'

# Conditional update
curl -X PUT 'http://localhost:8080/Organization/org-a/fhir/Observation?identifier=http://acme.org/obs|12345' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType": "Observation", "status": "final"}'

# Conditional delete
curl -X DELETE 'http://localhost:8080/Organization/org-a/fhir/Observation?identifier=http://acme.org/obs|12345' \
  -u root:<secret>
```

---

## Bundle Operations

```bash
curl -X POST 'http://localhost:8080/Organization/org-a/fhir/' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "request": {"method": "POST", "url": "Patient"},
      "resource": {
        "resourceType": "Patient",
        "birthDate": "2021-01-01",
        "meta": {
          "organization": {"id": "org-c", "resourceType": "Organization"}
        }
      }
    },
    {
      "request": {"method": "POST", "url": "Patient"},
      "resource": {"resourceType": "Patient", "birthDate": "2021-01-01"}
    }
  ]
}'
```

Or use org-based URLs in bundle:

```bash
curl -X POST 'http://localhost:8080/fhir' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "request": {"method": "PUT", "url": "/Organization/org-a/fhir/Patient/pt-1"},
      "resource": {"resourceType": "Patient", "birthDate": "2021-01-01"}
    },
    {
      "request": {"method": "PUT", "url": "/Organization/org-b/fhir/Patient/pt-2"},
      "resource": {"resourceType": "Patient", "birthDate": "2021-01-01"}
    }
  ]
}'
```

---

## Shared Resources

Resources accessible by nested organizations (read-only from children):

```bash
curl -X PUT 'http://localhost:8080/Organization/org-a/fhir/Practitioner/prac-1' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Practitioner",
  "meta": {
    "extension": [
      {"url": "https://aidbox.app/tenant-resource-mode", "valueString": "shared"}
    ]
  },
  "name": [{"family": "Smith", "given": ["John"]}]
}'
```

Now `org-b` and `org-c` can read this Practitioner, but can't update/delete it.

---

## AccessPolicy for OrgBAC Users

```bash
curl -X PUT 'http://localhost:8080/AccessPolicy/org-user-patient-access' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AccessPolicy",
  "id": "org-user-patient-access",
  "description": "User can access patients in their organization",
  "engine": "matcho",
  "matcho": {
    "params": {"resource/type": "Patient"},
    "request-method": "get",
    "user": {
      "meta": {
        "extension": {
          "$contains": {
            "url": "https://aidbox.app/tenant-organization-id",
            "value": {"Reference": {"id": ".params.organization/id"}}
          }
        }
      }
    }
  }
}'
```

---

## AidboxQuery with OrgBAC

Create `organization` param explicitly:

```bash
curl -X PUT 'http://localhost:8080/AidboxQuery/patients-in-org' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxQuery",
  "id": "patients-in-org",
  "params": {"organization": {"type": "string"}},
  "query": "SELECT * FROM patient pt WHERE pt.resource#>>'"'"'{meta,organization,id}'"'"' = {{params.organization}}",
  "count-query": "SELECT count(*) FROM patient pt WHERE pt.resource#>>'"'"'{meta,organization,id}'"'"' = {{params.organization}}",
  "type": "query"
}'
```

Execute:

```bash
curl 'http://localhost:8080/Organization/org-a/$query/patients-in-org' -u root:<secret>
# organization param is automatically set to org-a
```

---

## Special Operations

### $everything

```bash
curl 'http://localhost:8080/Organization/org-a/fhir/Patient/pt-1/$everything' -u root:<secret>
```

### Group-level Export

```bash
# Start export
curl 'http://localhost:8080/Organization/org-a/fhir/Group/grp-1/$export' -u root:<secret>

# Check status
curl 'http://localhost:8080/Organization/org-a/fhir/$export-status/<export-id>' -u root:<secret>

# Cancel
curl -X DELETE 'http://localhost:8080/Organization/org-a/fhir/$export-status/<export-id>' -u root:<secret>
```

### GraphQL (2503+)

```bash
curl -X POST 'http://localhost:8080/Organization/org-a/aidbox/$graphql' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"query": "{ PatientList { id name { family } } }"}'
```

---

## Limitations

### Not Supported Under OrgBAC API

- `_assoc` search parameter
- `_with` search parameter
- SubsSubscription resource (use AidboxSubscriptionTopic instead)

### Topic-Based Subscriptions (2509+)

AidboxSubscriptionTopic supports OrgBAC filtering:

```bash
curl -X PUT 'http://localhost:8080/AidboxTopicDestination/org-notifications' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxTopicDestination",
  "topic": "patient-changes",
  "kind": "webhook",
  "parameter": [
    {"name": "url", "valueUrl": "https://webhook.example.com/fhir"},
    {"name": "organizationHierarchy", "valueReference": {"reference": "Organization/org-a"}}
  ]
}'
```

---

## Common Issues

### Resource Not Visible

1. Check organization hierarchy (`partOf` references)
2. Verify user has correct organization extension
3. Check AccessPolicy matches organization path

### 403 Forbidden on Read

- Resource belongs to different organization branch
- Missing AccessPolicy for the operation

### Can't Update Shared Resource

Shared resources can only be updated from their root organization API, not from nested organization APIs.

---

## Init Bundle Example

```bash
curl -X POST 'http://localhost:8080/fhir' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "resource": {"resourceType": "Organization", "id": "tenant-1", "name": "Tenant 1"},
      "request": {"method": "PUT", "url": "Organization/tenant-1"}
    },
    {
      "resource": {
        "resourceType": "Organization",
        "id": "tenant-1-clinic",
        "name": "Tenant 1 Clinic",
        "partOf": {"reference": "Organization/tenant-1"}
      },
      "request": {"method": "PUT", "url": "Organization/tenant-1-clinic"}
    },
    {
      "resource": {
        "resourceType": "User",
        "id": "tenant-1-admin",
        "email": "admin@tenant1.com",
        "password": "secret",
        "meta": {
          "extension": [
            {
              "url": "https://aidbox.app/tenant-organization-id",
              "valueReference": {"reference": "Organization/tenant-1"}
            }
          ]
        }
      },
      "request": {"method": "PUT", "url": "User/tenant-1-admin"}
    },
    {
      "resource": {
        "resourceType": "AccessPolicy",
        "id": "tenant-user-access",
        "engine": "matcho",
        "matcho": {
          "uri": "#^/Organization/.user.meta.extension.0.value.Reference.id/fhir/.*$",
          "request-method": {"$one-of": ["get", "post", "put", "patch", "delete"]}
        }
      },
      "request": {"method": "PUT", "url": "AccessPolicy/tenant-user-access"}
    }
  ]
}'
```
