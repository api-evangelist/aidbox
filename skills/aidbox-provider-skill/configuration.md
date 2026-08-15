# Configuration Reference

## Recommended Environment Variables

### Standard Aidbox
```yaml
# Required
BOX_ADMIN_PASSWORD: <admin-password>
BOX_DB_HOST: postgres
BOX_DB_PORT: '5432'
BOX_DB_DATABASE: aidbox
BOX_DB_USER: aidbox
BOX_DB_PASSWORD: <pg-password>
BOX_WEB_BASE_URL: <base-url>
BOX_WEB_PORT: 8888
BOX_ROOT_CLIENT_SECRET: <client-secret>

# FHIR Configuration
BOX_BOOTSTRAP_FHIR_PACKAGES: hl7.fhir.r4.core#4.0.1
BOX_FHIR_COMPLIANT_MODE: 'true'
BOX_FHIR_SCHEMA_VALIDATION: 'true'
BOX_FHIR_CORRECT_AIDBOX_FORMAT: 'true'
BOX_FHIR_BUNDLE_EXECUTION_VALIDATION_MODE: limited

# Search
BOX_FHIR_SEARCH_AUTHORIZE_INLINE_REQUESTS: 'true'
BOX_FHIR_SEARCH_CHAIN_SUBSELECT: 'true'
BOX_FHIR_SEARCH_COMPARISONS: 'true'
BOX_SEARCH_INCLUDE_CONFORMANT: 'true'

# Terminology
BOX_FHIR_TERMINOLOGY_ENGINE: hybrid
BOX_FHIR_TERMINOLOGY_SERVICE_BASE_URL: https://tx.health-samurai.io/fhir
BOX_FHIR_TERMINOLOGY_ENGINE_HYBRID_EXTERNAL_TX_SERVER: https://tx.health-samurai.io/fhir

# Security
BOX_SECURITY_AUDIT_LOG_ENABLED: 'true'
BOX_SECURITY_DEV_MODE: 'true'  # ONLY for development!
BOX_SETTINGS_MODE: read-write

# Other
BOX_FHIR_CREATEDAT_URL: https://aidbox.app/ex/createdAt
BOX_COMPATIBILITY_VALIDATION_JSON__SCHEMA_REGEX: '#{:fhir-datetime}'
BOX_MODULE_SDC_STRICT_ACCESS_CONTROL: 'true'
```

### Aidbox with MCP (AI Integration)
```yaml
# Add to standard config:
BOX_MODULE_MCP_SERVER_ENABLED: 'true'
BOX_INIT_BUNDLE: https://storage.googleapis.com/aidbox-public/mcp/mcp-init-bundle.json
```

### Aidbox R6
```yaml
# Change FHIR packages:
BOX_BOOTSTRAP_FHIR_PACKAGES: hl7.fhir.r6.core#6.0.0-ballot3
BOX_FHIR_TERMINOLOGY_SERVICE_BASE_URL: https://tx3.health-samurai.io/fhir
BOX_FHIR_TERMINOLOGY_ENGINE_HYBRID_EXTERNAL_TX_SERVER: https://tx3.health-samurai.io/fhir
```

---

## Key Settings Explained

### FHIR Mode
| Setting | Description | Recommended |
|---------|-------------|-------------|
| `BOX_FHIR_COMPLIANT_MODE` | FHIR compliance (adds Bundle.link.url base) | `true` |
| `BOX_FHIR_SCHEMA_VALIDATION` | Enable FHIRSchema validation | `true` |
| `BOX_FHIR_CORRECT_AIDBOX_FORMAT` | Auto-correct Aidbox format issues | `true` |

### Security
| Setting | Description | Recommended |
|---------|-------------|-------------|
| `BOX_SECURITY_DEV_MODE` | Allows `engine: allow` policies | `false` in production |
| `BOX_SECURITY_AUDIT_LOG_ENABLED` | Enable AuditEvent logging | `true` |
| `BOX_SETTINGS_MODE` | `read-write` or `read-only` | `read-write` |

### Search Performance
| Setting | Description | Recommended |
|---------|-------------|-------------|
| `BOX_FHIR_SEARCH_DEFAULT_PARAMS_COUNT` | Default page size | `100` |
| `BOX_FHIR_SEARCH_DEFAULT_PARAMS_TOTAL` | Count method: `none`, `estimate`, `accurate` | `accurate` |
| `BOX_FHIR_SEARCH_CHAIN_SUBSELECT` | Use subselects for chaining | `true` |

### Database
| Setting | Description | Recommended |
|---------|-------------|-------------|
| `BOX_DB_POOL_MAXIMUM_POOL_SIZE` | Connection pool size | `8` |
| `BOX_DB_RO_REPLICA_ENABLED` | Enable read replica | `false` (enable for scale) |
| `BOX_VIEW_DEFINITION_SCHEMA` | Schema for materialized views | `sof` |

---

## Init Bundle

Set `BOX_INIT_BUNDLE` to URL or local path:

```yaml
# As environment variable
BOX_INIT_BUNDLE: /config/init-bundle.yaml
# or
BOX_INIT_BUNDLE: https://example.com/init-bundle.json
```

### Init Bundle Example

```yaml
resourceType: Bundle
type: transaction
entry:
  # Create application Client
  - resource:
      resourceType: Client
      id: my-app
      secret: secret123
      grant_types: ["client_credentials"]
    request:
      method: PUT
      url: Client/my-app

  # AccessPolicy for the Client
  - resource:
      resourceType: AccessPolicy
      id: my-app-policy
      engine: matcho
      matcho:
        uri: "#^/fhir/.*$"
        request-method: {$one-of: [get, post, put, patch, delete]}
      link:
        - {resourceType: Client, id: my-app}
    request:
      method: PUT
      url: AccessPolicy/my-app-policy

  # Load Implementation Guide
  - resource:
      resourceType: FHIRPackage
      id: us-core
      name: hl7.fhir.us.core
      version: "6.1.0"
    request:
      method: PUT
      url: FHIRPackage/us-core

  # Custom SearchParameter
  - resource:
      resourceType: SearchParameter
      id: patient-mrn
      url: "http://example.org/SearchParameter/patient-mrn"
      name: mrn
      status: active
      code: mrn
      base: [Patient]
      type: token
      expression: "Patient.identifier.where(system='http://example.org/mrn')"
    request:
      method: PUT
      url: /fhir/SearchParameter/patient-mrn

  # ViewDefinition for analytics
  - resource:
      resourceType: ViewDefinition
      id: patient_view
      name: patient_view
      status: draft
      resource: Patient
      select:
        - column:
            - {name: id, path: id}
            - {name: gender, path: gender}
    request:
      method: PUT
      url: ViewDefinition/patient_view
```

---

## FHIR Package Configuration

### Loading Packages at Startup
```yaml
BOX_BOOTSTRAP_FHIR_PACKAGES: "hl7.fhir.r4.core#4.0.1,hl7.fhir.us.core#6.1.0"
```

### Custom NPM Registry
```yaml
BOX_FHIR_NPM_PACKAGE_REGISTRY: "https://custom-registry.example.com"
```

### Local Packages
Mount to `/srv/aidbox-fhir-packages`:
```bash
# Format: {package-name}#{version}.tgz
hl7.fhir.r4.core#4.0.1.tgz
hl7.fhir.us.core#6.1.0.tgz
```

---

## Database Configuration

### Primary Database
```yaml
BOX_DB_HOST: postgres
BOX_DB_PORT: 5432
BOX_DB_DATABASE: aidbox
BOX_DB_USER: aidbox
BOX_DB_PASSWORD: <password>
BOX_DB_POOL_MAXIMUM_POOL_SIZE: 8
```

### Read Replica (for scaling reads)
```yaml
BOX_DB_RO_REPLICA_ENABLED: true
BOX_DB_RO_REPLICA_HOST: postgres-replica
BOX_DB_RO_REPLICA_PORT: 5432
BOX_DB_RO_REPLICA_DATABASE: aidbox
BOX_DB_RO_REPLICA_USER: aidbox
BOX_DB_RO_REPLICA_PASSWORD: <password>
BOX_DB_RO_REPLICA_POOL_MAXIMUM_POOL_SIZE: 8
```

---

## Validation Configuration

### Skip Reference Validation (for imports)
```yaml
BOX_FHIR_VALIDATION_SKIP_REFERENCE: true
```

### Validation Mode for Bundles
```yaml
# Options: strict, limited
BOX_FHIR_BUNDLE_EXECUTION_VALIDATION_MODE: limited
```

---

## Common Configurations

### Production Checklist
```yaml
# Security
BOX_SECURITY_DEV_MODE: 'false'           # Disable dev mode!
BOX_SECURITY_AUDIT_LOG_ENABLED: 'true'   # Enable audit logging

# Performance
BOX_FHIR_SEARCH_DEFAULT_PARAMS_TOTAL: 'estimate'  # Faster counts
BOX_DB_POOL_MAXIMUM_POOL_SIZE: 16        # Increase pool size

# Disable settings API in production
BOX_SETTINGS_MODE: 'read-only'
```

### Development Configuration
```yaml
BOX_SECURITY_DEV_MODE: 'true'            # Allow engine: allow
BOX_SETTINGS_MODE: 'read-write'          # Allow settings changes
BOX_OBSERVABILITY_STDOUT_LOG_LEVEL: 'debug'  # Verbose logging
```
