# Devin Requirement Spec
> Fill in this file for EACH new component you want Devin to build.
> Attach this file alongside `devin/master-template.md` when giving Devin a task.
>
> How to give to Devin:
> "Read devin/master-template.md for project standards, then implement the
>  requirement described in this file. Complete fully and autonomously."

---

## COMPONENT NAME

```
[Short name for this component, e.g. KafkaOrderConsumer / PaymentApiAdapter / FirestoreRepository]
```

---

## WHAT TO BUILD

```
[Describe the component in clear, specific terms. Include all functional requirements.]

Example:
Build a Kafka consumer component that:
- Listens to topic: ${KAFKA_INPUT_TOPIC}
- Connects using bootstrap servers: ${KAFKA_BOOTSTRAP_SERVERS}
- Authenticates with SASL/SSL using credentials from environment variables
- Deserializes each message as JSON into the OrderEvent schema (see below)
- Passes each message to the ProcessOrderUseCase
- On success: commits the Kafka offset
- On failure: publishes the original message to dead-letter topic ${KAFKA_DLT_TOPIC}
               and logs the error with full context including message ID and trace ID
```

---

## INPUT SCHEMA

```
[Describe the input message, request body, or event format]

Example:
{
  "order_id":    "string (UUID)",
  "customer_id": "string (UUID)",
  "items": [
    {
      "sku":      "string",
      "quantity": "integer",
      "price":    "float"
    }
  ],
  "timestamp": "ISO 8601 UTC string"
}
```

---

## OUTPUT / SIDE EFFECTS

```
[Describe what the component produces, persists, or publishes]

Example:
- Order record written to Firestore collection: ${FIRESTORE_ORDERS_COLLECTION}
- Confirmation event published to Kafka topic: ${KAFKA_OUTPUT_TOPIC}
- Structured log entry written for each processed message
```

---

## ENVIRONMENT VARIABLES SPECIFIC TO THIS COMPONENT

```
[List only the variables specific to this component — the global ones are in master-template.md]

| Variable Name              | Description                        |
|----------------------------|------------------------------------|
| [COMPONENT_SPECIFIC_VAR_1] | [e.g. Kafka input topic name]      |
| [COMPONENT_SPECIFIC_VAR_2] | [e.g. Kafka DLT topic name]        |
| [COMPONENT_SPECIFIC_VAR_3] | [e.g. Firestore collection name]   |
| [ADD MORE AS NEEDED]       |                                    |
```

---

## NON-FUNCTIONAL REQUIREMENTS

```
- Throughput:    [e.g., handle up to 1000 messages/sec]
- Latency:       [e.g., p99 processing time < 200ms]
- Availability:  [e.g., auto-restart on crash, no message loss]
- Idempotency:   [e.g., safe to process the same message twice]
```

---

## BRANCH NAME

```
[e.g., feature/kafka-order-consumer]
```

---

## EXPLICITLY OUT OF SCOPE

```
[List what Devin should NOT build for this task — prevents scope creep]

- [e.g., Authentication logic — handled by a separate component]
- [e.g., Frontend or UI]
- [e.g., CI/CD pipeline changes]
- [e.g., Database schema migrations]
```

---

## ADDITIONAL NOTES / ASSUMPTIONS

```
[Any other context Devin needs. Leave blank if none.]

Example:
- The ProcessOrderUseCase already exists at src/application/process_order_use_case.py
- Use the existing KafkaAdapterBase class in src/adapters/kafka_base.py as the parent class
- Follow the same pattern as the existing InventoryConsumer in src/adapters/inventory_consumer.py
```
