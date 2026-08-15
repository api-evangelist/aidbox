# Getting Started

## Quick Start

**Step 1: Download (if needed) and start**
```bash
test -f docker-compose.yml || curl -JO https://aidbox.app/runme
docker compose up -d
```

This downloads docker-compose.yml and starts:
- **Aidbox** on http://localhost:8080
- **PostgreSQL** database

**Step 2: Wait for Aidbox to start**
```bash
until curl -sf http://localhost:8080/health > /dev/null 2>&1; do sleep 2; done
```
This polls healthcheck endpoint every 2 seconds until Aidbox is ready.


---

## ⚠️ CRITICAL: First Launch Activation Required

**Aidbox will NOT work without activation.** On first launch, you MUST activate the license before making any API calls.

### Step 1: Wait for Aidbox to Start

After `docker compose up`, wait until you see in logs:
```
Aidbox started
```
Or check: `curl http://localhost:8080/health` returns `healthy`.

### Step 2: Activate License (Choose One Method)

#### Method A: AidboxID (Recommended for Development)

1. Open http://localhost:8080 in browser
2. You will see the activation screen
3. Click **"Sign in with AidboxID"**
4. Create account or sign in at https://aidbox.app
5. Aidbox will automatically receive a development license
6. **Only after activation** will the Admin UI and API become functional

For automated setup, add environment variables to docker-compose.yml:
```yaml
environment:
  AIDBOX_CLIENT_ID: <your-aidbox-id-client-id>
  AIDBOX_CLIENT_SECRET: <your-aidbox-id-client-secret>
```
Get credentials at https://aidbox.app → Settings → Auth Clients.

#### Method B: License Key (Production)

Set `BOX_LICENSE` environment variable in docker-compose.yml:
```yaml
environment:
  BOX_LICENSE: <your-license-key>
```
Get license at https://aidbox.app → Licenses.

### Step 3: Get Root Client Secret

After activation, get the root client secret:

```bash
docker compose config | grep BOX_ROOT_CLIENT_SECRET | awk '{print $2}'
```

This returns the secret value (e.g. `kx1BGnjiRH`) - use it for API calls.

**Note**: Without activation, Aidbox runs in limited mode and API calls will fail.

---

## Authentication

### Basic Auth (Development)

Use `root` client with the secret:

```bash
# Get secret
secret=$(docker compose config | grep BOX_ROOT_CLIENT_SECRET | awk '{print $2}')

# Use it
curl -u root:$secret http://localhost:8080/fhir/Patient
```

### Client Credentials (Production)

1. Create Client:
```bash
curl -X PUT 'http://localhost:8080/Client/my-app' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"Client","id":"my-app","secret":"my-secret","grant_types":["client_credentials"]}'
```

2. Create AccessPolicy:
```bash
curl -X PUT 'http://localhost:8080/AccessPolicy/my-app-policy' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{"resourceType":"AccessPolicy","id":"my-app-policy","engine":"matcho","matcho":{"uri":"#^/fhir/.*$"},"link":[{"resourceType":"Client","id":"my-app"}]}'
```

3. Get token:
```bash
curl -X POST 'http://localhost:8080/auth/token' \
  -d 'grant_type=client_credentials&client_id=my-app&client_secret=my-secret'
```

4. Use token:
```bash
curl 'http://localhost:8080/fhir/Patient' \
  -H 'Authorization: Bearer <access_token>'
```

## Custom Configuration

### With License (Production)

Create `docker-compose.yml`:
```yaml
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: aidbox
      POSTGRES_PASSWORD: <pg-password>
      POSTGRES_DB: aidbox
    volumes:
      - postgres_data:/var/lib/postgresql/data

  aidbox:
    image: healthsamurai/aidboxone:stable
    depends_on:
      - db
    ports:
      - "8080:8080"
    environment:
      BOX_LICENSE: <your-license>
      BOX_WEB_BASE_URL: http://localhost:8080
      BOX_ADMIN_PASSWORD: <admin-password>
      BOX_ROOT_CLIENT_SECRET: <client-secret>
      BOX_DB_HOST: db
      BOX_DB_PORT: 5432
      BOX_DB_DATABASE: aidbox
      BOX_DB_USER: aidbox
      BOX_DB_PASSWORD: <pg-password>
      BOX_FHIR_COMPLIANT_MODE: "true"
      BOX_BOOTSTRAP_FHIR_PACKAGES: "hl7.fhir.r4.core#4.0.1"

volumes:
  postgres_data:
```

### Image Tags

| Tag | Use Case |
|-----|----------|
| `edge` | Latest development (testing) |
| `stable` | Production recommended |
| `2024.1.0` | Specific version (reproducible) |

## Verify Installation

```bash
# Health check
curl http://localhost:8080/health

# Get capability statement
curl http://localhost:8080/fhir/metadata

# Create test patient
curl -X POST http://localhost:8080/fhir/Patient \
  -H "Content-Type: application/json" \
  -u root:<secret> \
  -d '{"resourceType": "Patient", "name": [{"family": "Test"}]}'
```

## Next Steps

- Load sample data: see [bulk-operations.md](bulk-operations.md#sample-data-synthea)
- Configure access control: see [access-policy.md](access-policy.md)
- Set up AI integration: see [mcp-setup.md](mcp-setup.md)
