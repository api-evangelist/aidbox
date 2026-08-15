# Subscriptions Reference

Aidbox provides topic-based subscriptions for real-time notifications when FHIR resources change.

## Quick Start

**Recommendation**: Use **Aidbox Topic-Based Subscriptions** for all new implementations.

| Implementation | Delivery | Channels |
|---------------|----------|----------|
| Aidbox Topic-Based | At-least-once / At-most-once | Kafka, Webhook, Pub/Sub, NATS, AMQP, EventBridge |
| FHIR Topic-Based | At-least-once | REST-hook |

---

## Aidbox Topic-Based Subscriptions

### Step 1: Create AidboxSubscriptionTopic

Define what events to subscribe to:

```bash
curl -X POST 'http://localhost:8080/fhir/AidboxSubscriptionTopic' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxSubscriptionTopic",
  "url": "http://example.org/SubscriptionTopic/patient-changes",
  "status": "active",
  "trigger": [
    {
      "resource": "Patient",
      "supportedInteraction": ["create", "update", "delete"],
      "fhirPathCriteria": "active = true"
    }
  ]
}'
```

**Trigger properties**:
- `resource` - FHIR resource type (required)
- `supportedInteraction` - create | update | delete
- `fhirPathCriteria` - FHIRPath filter (supports `%current` and `%previous`)

### Step 2: Create AidboxTopicDestination

Define where to send notifications.

#### Kafka Destination
```bash
curl -X POST 'http://localhost:8080/fhir/AidboxTopicDestination' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxTopicDestination",
  "meta": {
    "profile": ["http://aidbox.io/StructureDefinition/aidboxtopicdestination-kafka-at-least-once"]
  },
  "status": "active",
  "topic": "http://example.org/SubscriptionTopic/patient-changes",
  "kind": "kafka-at-least-once",
  "content": "full-resource",
  "parameter": [
    {"name": "kafkaTopic", "valueString": "patient-events"},
    {"name": "bootstrapServers", "valueString": "kafka:9092"}
  ]
}'
```

#### Webhook Destination
```bash
curl -X POST 'http://localhost:8080/fhir/AidboxTopicDestination' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxTopicDestination",
  "meta": {
    "profile": ["http://aidbox.io/StructureDefinition/aidboxtopicdestination-webhook-at-least-once"]
  },
  "status": "active",
  "topic": "http://example.org/SubscriptionTopic/patient-changes",
  "kind": "webhook-at-least-once",
  "content": "full-resource",
  "parameter": [
    {"name": "endpoint", "valueUrl": "https://myapp.com/webhooks/fhir"},
    {"name": "timeout", "valueInteger": 30}
  ]
}'
```

#### GCP Pub/Sub Destination
```bash
curl -X POST 'http://localhost:8080/fhir/AidboxTopicDestination' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxTopicDestination",
  "meta": {
    "profile": ["http://aidbox.io/StructureDefinition/aidboxtopicdestination-gcp-pubsub-at-least-once"]
  },
  "status": "active",
  "topic": "http://example.org/SubscriptionTopic/patient-changes",
  "kind": "gcp-pubsub-at-least-once",
  "content": "full-resource",
  "parameter": [
    {"name": "projectId", "valueString": "my-gcp-project"},
    {"name": "topicId", "valueString": "fhir-events"}
  ]
}'
```

#### AWS EventBridge Destination
```bash
curl -X POST 'http://localhost:8080/fhir/AidboxTopicDestination' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "resourceType": "AidboxTopicDestination",
  "meta": {
    "profile": ["http://aidbox.io/StructureDefinition/aidboxtopicdestination-aws-eventbridge"]
  },
  "status": "active",
  "topic": "http://example.org/SubscriptionTopic/patient-changes",
  "kind": "aws-eventbridge",
  "content": "full-resource",
  "parameter": [
    {"name": "eventBusArn", "valueString": "arn:aws:events:us-east-1:123456789:event-bus/my-bus"},
    {"name": "source", "valueString": "aidbox"}
  ]
}'
```

---

## Destination Kinds

| Kind | Guarantee | Profile |
|------|-----------|---------|
| `kafka-at-least-once` | At-least-once | `aidboxtopicdestination-kafka-at-least-once` |
| `kafka-best-effort` | At-most-once | `aidboxtopicdestination-kafka-best-effort` |
| `webhook-at-least-once` | At-least-once | `aidboxtopicdestination-webhook-at-least-once` |
| `gcp-pubsub-at-least-once` | At-least-once | `aidboxtopicdestination-gcp-pubsub-at-least-once` |
| `nats` | At-most-once | `aidboxtopicdestination-nats` |
| `amqp` | Depends on config | `aidboxtopicdestination-amqp` |
| `aws-eventbridge` | At-least-once | `aidboxtopicdestination-aws-eventbridge` |

---

## Content Types

| Value | Description |
|-------|-------------|
| `full-resource` | Complete resource in notification (default) |
| `id-only` | Only reference to resource |
| `empty` | Just notification event metadata |

---

## Notification Shape

Notifications are FHIR Bundles with `history` type:

```json
{
  "resourceType": "Bundle",
  "type": "history",
  "timestamp": "2024-10-03T10:07:55Z",
  "entry": [
    {
      "resource": {
        "resourceType": "AidboxSubscriptionStatus",
        "status": "active",
        "type": "event-notification",
        "notificationEvent": [
          {
            "eventNumber": 1,
            "focus": {
              "reference": "Patient/pt-123"
            }
          }
        ],
        "topic": "http://example.org/SubscriptionTopic/patient-changes",
        "topic-destination": {
          "reference": "AidboxTopicDestination/kafka-destination"
        }
      }
    },
    {
      "resource": {
        "resourceType": "Patient",
        "id": "pt-123"
      },
      "fullUrl": "http://aidbox/fhir/Patient/pt-123",
      "request": {
        "method": "POST",
        "url": "/fhir/Patient"
      }
    }
  ]
}
```

---

## FHIRPath Criteria Examples

```
# Only active patients
fhirPathCriteria: "active = true"

# Compare current vs previous state
fhirPathCriteria: "%current.status != %previous.status"

# Only completed QuestionnaireResponses
fhirPathCriteria: "status = 'completed' or status = 'amended'"
```

---

## Managing Subscriptions

### Stop Subscription
Delete the `AidboxTopicDestination` resource:
```bash
curl -X DELETE 'http://localhost:8080/fhir/AidboxTopicDestination/my-destination' -u root:<secret>
```

### Check Subscription Status
```bash
curl 'http://localhost:8080/fhir/AidboxTopicDestination/my-destination' -u root:<secret>
```

---

## Choosing the Right Channel

| Use Case | Recommended Channel |
|----------|-------------------|
| Simple webhook integration | Webhook |
| High-throughput processing | Kafka |
| AWS ecosystem | EventBridge |
| GCP ecosystem | Pub/Sub |
| Real-time streaming | NATS |
| Enterprise messaging | AMQP (RabbitMQ) |

---

## Troubleshooting

### Subscription Not Triggering
1. Check topic status is `active`
2. Verify `fhirPathCriteria` matches your data
3. Ensure destination profile is correct
4. Check Aidbox logs for errors

### Testing FHIRPath Criteria
```bash
curl -X POST 'http://localhost:8080/$fhirpath' \
  -H 'Content-Type: application/json' \
  -u root:<secret> \
  -d '{
  "expression": "active = true",
  "resource": {
    "resourceType": "Patient",
    "active": true
  }
}'
```
