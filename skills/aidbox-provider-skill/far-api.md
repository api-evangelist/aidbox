# FHIR Artifact Registry (FAR) Reference

FAR is Aidbox's centralized system for storing and managing FHIR canonical resources and packages.

## What FAR Stores

| Resource Type | Purpose |
|--------------|---------|
| StructureDefinition | Profiles, extensions, custom resources |
| CodeSystem | Terminology definitions |
| ValueSet | Code groupings |
| ConceptMap | Terminology mappings |
| SearchParameter | Custom search capabilities |

---

## Loading FHIR Packages

### Via Environment Variable (Startup)
```bash
BOX_BOOTSTRAP_FHIR_PACKAGES: "hl7.fhir.r4.core#4.0.1,hl7.fhir.us.core#6.1.0"
```

### Via Init Bundle
```bash
curl -X POST 'http://localhost:8080/fhir' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    {
      "resource": {
        "resourceType": "FHIRPackage",
        "id": "us-core",
        "name": "hl7.fhir.us.core",
        "version": "6.1.0"
      },
      "request": {"method": "PUT", "url": "FHIRPackage/us-core"}
    }
  ]
}'
```

### Via REST API
```bash
curl -X PUT 'http://localhost:8080/fhir/FHIRPackage/us-core' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "FHIRPackage",
  "id": "us-core",
  "name": "hl7.fhir.us.core",
  "version": "6.1.0"
}'
```

### Via Local Filesystem
Mount packages to `/srv/aidbox-fhir-packages` in container:
```bash
# Format: {package-name}#{version}.tgz
hl7.fhir.r4.core#4.0.1.tgz
hl7.fhir.us.core#6.1.0.tgz
```

---

## Accessing Canonical Resources

### REST API Endpoints
```bash
curl 'http://localhost:8080/fhir/StructureDefinition' -u root:<secret>
curl 'http://localhost:8080/fhir/CodeSystem' -u root:<secret>
curl 'http://localhost:8080/fhir/ValueSet' -u root:<secret>
curl 'http://localhost:8080/fhir/ConceptMap' -u root:<secret>
curl 'http://localhost:8080/fhir/SearchParameter' -u root:<secret>
```

### Search Examples
```bash
# Find profiles for Patient
curl 'http://localhost:8080/fhir/StructureDefinition?type=Patient' -u root:<secret>

# Find all US Core profiles
curl 'http://localhost:8080/fhir/StructureDefinition?url:below=http://hl7.org/fhir/us/core' -u root:<secret>

# Find ValueSet by name
curl 'http://localhost:8080/fhir/ValueSet?name=us-core-condition-code' -u root:<secret>
```

---

## Custom Resources

Create custom resource types via StructureDefinition:

```bash
curl -X POST 'http://localhost:8080/fhir/StructureDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "StructureDefinition",
  "id": "MyCustomResource",
  "url": "http://example.org/StructureDefinition/MyCustomResource",
  "name": "MyCustomResource",
  "status": "active",
  "kind": "resource",
  "abstract": false,
  "type": "MyCustomResource",
  "baseDefinition": "http://hl7.org/fhir/StructureDefinition/DomainResource",
  "derivation": "specialization",
  "differential": {
    "element": [
      {
        "id": "MyCustomResource",
        "path": "MyCustomResource",
        "short": "My custom resource"
      },
      {
        "id": "MyCustomResource.customField",
        "path": "MyCustomResource.customField",
        "min": 0,
        "max": "1",
        "type": [{"code": "string"}]
      },
      {
        "id": "MyCustomResource.patient",
        "path": "MyCustomResource.patient",
        "min": 1,
        "max": "1",
        "type": [{"code": "Reference", "targetProfile": ["http://hl7.org/fhir/StructureDefinition/Patient"]}]
      }
    ]
  }
}'
```

Then use it:
```bash
curl -X POST 'http://localhost:8080/fhir/MyCustomResource' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "MyCustomResource",
  "customField": "some value",
  "patient": {"reference": "Patient/pt-123"}
}'
```

---

## Extensions

### Define Extension
```bash
curl -X POST 'http://localhost:8080/fhir/StructureDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "StructureDefinition",
  "url": "http://example.org/StructureDefinition/patient-occupation",
  "name": "PatientOccupation",
  "status": "active",
  "kind": "complex-type",
  "abstract": false,
  "context": [
    {"type": "element", "expression": "Patient"}
  ],
  "type": "Extension",
  "baseDefinition": "http://hl7.org/fhir/StructureDefinition/Extension",
  "derivation": "constraint",
  "differential": {
    "element": [
      {
        "id": "Extension.value[x]",
        "path": "Extension.value[x]",
        "type": [{"code": "string"}]
      }
    ]
  }
}'
```

### Use Extension
```json
{
  "resourceType": "Patient",
  "extension": [
    {
      "url": "http://example.org/StructureDefinition/patient-occupation",
      "valueString": "Engineer"
    }
  ]
}
```

---

## Configuration

### Package Registry
```bash
# Default: https://fs.get-ig.org/pkgs
BOX_FHIR_NPM_PACKAGE_REGISTRY: "https://custom-registry.com"
```

### Root Package
```bash
# Custom resources stored here
BOX_ROOT_FHIR_PACKAGE: "app.aidbox.main#0.0.1"
```

---

## Key Features

### Automatic Dependency Resolution
When loading a package, all dependencies are automatically resolved and loaded.

### Pinning
References inside canonicals are pinned to exact dependency versions.

### Tree-Shaking
Only referenced canonical dependencies are installed from dependent packages.

---

## Web UI

Access via AidboxUI -> FHIR Packages:
- View installed packages
- Import new packages from registry
- Delete packages
- Browse package contents

---

## Validation

### Enable FHIRSchema Validation

```yaml
BOX_FHIR_SCHEMA_VALIDATION: 'true'
```

### Validation Modes

| Setting | Description |
|---------|-------------|
| `BOX_FHIR_SCHEMA_VALIDATION: 'true'` | Enable validation |
| `BOX_FHIR_BUNDLE_EXECUTION_VALIDATION_MODE: 'limited'` | Lighter validation for bundles |
| `Skip-Validation: reference` header | Skip reference validation per request |

### Skip Validation for Import

```bash
curl -X POST 'http://localhost:8080/$import' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "inputFormat": "application/fhir+ndjson",
  "mode": "bulk",
  "input": [{"url": "..."}]
}'
```

`mode: "bulk"` skips all validation.

---

## Extension Gotchas

### Problem: "Unknown extension" Error

**Solution:** Load StructureDefinition for the extension:

```bash
curl -X POST 'http://localhost:8080/fhir/StructureDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "StructureDefinition",
  "url": "http://example.org/StructureDefinition/patient-occupation",
  "name": "PatientOccupation",
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

### Problem: Can't Create Extension for User/Role Resource

Aidbox resources (User, Role, Client) are not standard FHIR. Use `meta.extension` instead:

```bash
curl -X PUT 'http://localhost:8080/User/user-1' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "User",
  "email": "user@example.com",
  "meta": {
    "extension": [
      {
        "url": "http://example.org/organization-id",
        "valueString": "org-123"
      }
    ]
  }
}'
```

### Problem: Nested Extension Not Working

Complex extensions need proper structure:

```bash
curl -X POST 'http://localhost:8080/fhir/StructureDefinition' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "StructureDefinition",
  "url": "http://example.org/StructureDefinition/address-details",
  "name": "AddressDetails",
  "status": "active",
  "kind": "complex-type",
  "abstract": false,
  "context": [{"type": "element", "expression": "Patient.address"}],
  "type": "Extension",
  "baseDefinition": "http://hl7.org/fhir/StructureDefinition/Extension",
  "derivation": "constraint",
  "differential": {
    "element": [
      {"id": "Extension", "path": "Extension"},
      {"id": "Extension.extension:latitude", "path": "Extension.extension", "sliceName": "latitude", "min": 0, "max": "1"},
      {"id": "Extension.extension:latitude.url", "path": "Extension.extension.url", "fixedUri": "latitude"},
      {"id": "Extension.extension:latitude.value[x]", "path": "Extension.extension.value[x]", "type": [{"code": "decimal"}]},
      {"id": "Extension.extension:longitude", "path": "Extension.extension", "sliceName": "longitude", "min": 0, "max": "1"},
      {"id": "Extension.extension:longitude.url", "path": "Extension.extension.url", "fixedUri": "longitude"},
      {"id": "Extension.extension:longitude.value[x]", "path": "Extension.extension.value[x]", "type": [{"code": "decimal"}]}
    ]
  }
}'
```

### Problem: Search by Extension Not Working

Create SearchParameter:

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

### Problem: Extension on Primitive (e.g., birthDate)

Use underscore property:

```json
{
  "resourceType": "Patient",
  "birthDate": "1990-01-15",
  "_birthDate": {
    "extension": [
      {
        "url": "http://example.org/birthDateEstimated",
        "valueBoolean": true
      }
    ]
  }
}
```

---

## FHIRSchema Null Values (2405+)

FHIRSchema mode throws errors on `null` values. Strip them before sending:

```javascript
// JavaScript
const clean = JSON.parse(JSON.stringify(resource, (k, v) => v === null ? undefined : v));
```

---

## Troubleshooting

### Package Not Loading
1. Check package name and version format: `name#version`
2. Verify package exists in registry
3. Check dependencies are available
4. Look in Aidbox logs for errors

### Custom Resource Not Working
1. Ensure StructureDefinition has `kind: resource`
2. Set `derivation: specialization`
3. Base on `DomainResource` or `Resource`
4. Restart Aidbox after creating StructureDefinition

### Profile Not Validating
1. Check `derivation: constraint` (not `specialization`)
2. Ensure profile URL is referenced in `meta.profile`
3. Verify StructureDefinition loaded correctly

### Find Where a Canonical Came From
```bash
curl 'http://localhost:8080/fhir/StructureDefinition?url=http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient' -u root:<secret>
```
Check `meta.source` for package origin.
