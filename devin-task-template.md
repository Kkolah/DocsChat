# Devin Task Template
> Fill in this template and paste it directly into Devin's task input.
> Devin will plan, write code, write tests, run them, fix errors, and complete the task autonomously.

---

## SECTION 1: Task Description

```
You are an expert software engineer working on a Google Cloud ADK project 
using Hexagonal Architecture (Ports & Adapters).

Complete the following task fully and autonomously:
- Do NOT ask clarifying questions
- If anything is unclear, make a reasonable assumption and document it in a code comment
- Write all implementation code
- Write unit tests and integration tests
- Run the tests and fix any failures
- Do not stop until all tests pass

### Task
[Describe what needs to be built in 1-3 sentences]
Example: Build a Kafka consumer component that listens to an input topic, 
processes messages through the domain layer, and publishes results to Firestore.
```

---

## SECTION 2: Architecture & Code Standards

```
### Architecture
- Hexagonal Architecture (Ports & Adapters)
- Core domain logic must never depend on infrastructure
- Define a Port (interface) for every external system interaction
- Implement an Adapter for each external system

### Project Structure
src/
  domain/         # Entities, value objects, pure business logic
  ports/          # Inbound + outbound interfaces
  adapters/       # Implementations: Kafka, DB, HTTP, GCP services
  application/    # Use cases / application services
  config/         # Configuration loaded from environment variables
  main.py         # Entrypoint (adjust extension for your language)
tests/
  unit/           # Unit tests — mock all external dependencies
  integration/    # Integration tests — test adapters against real/emulated services

### Code Standards
- SOLID principles, clean code, meaningful names
- No hardcoded secrets or config — all from environment variables
- Structured JSON logging with correlation IDs
- Retry with exponential backoff for transient failures
- Validate and sanitize all external inputs
- Async/await throughout (if applicable to language)
- All dates in UTC ISO 8601 format

### Google Cloud ADK
- Use Google Cloud ADK idioms and best practices
- Use Application Default Credentials (ADC) for GCP authentication
- Use ADK-native clients for GCP services (Pub/Sub, Firestore, BigQuery, etc.)
- Implement proper agent lifecycle hooks: init, run, shutdown
- Use ADK's built-in observability and tracing

### Testing Requirements
- Unit tests for all domain logic and use cases
- Integration tests for all adapters
- Mock all external dependencies in unit tests
- All tests must pass before task is considered complete
- Test runner: [e.g., pytest / mvn test / npm test]
```

---

## SECTION 3: Environment & Configuration

```
### Runtime Environment
- Language: [e.g., Python 3.11 / Java 17 / Node.js 20]
- Framework: [e.g., FastAPI / Spring Boot / Express]
- Cloud Platform: Google Cloud Platform
- ADK Version: [e.g., google-cloud-adk 1.x]
- Package manager: [e.g., pip / maven / npm]

### Environment Variables
All secrets and config are injected via environment variables.
Never hardcode these values. Reference them as os.environ or equivalent.

| Variable Name                   | Description                          |
|---------------------------------|--------------------------------------|
| GOOGLE_CLOUD_PROJECT            | GCP Project ID                       |
| GOOGLE_APPLICATION_CREDENTIALS  | Path to service account key (if set) |
| [YOUR_VAR_1]                    | [Description, e.g. Kafka broker URL] |
| [YOUR_VAR_2]                    | [e.g. Kafka input topic name]        |
| [YOUR_VAR_3]                    | [e.g. Kafka DLT topic name]          |
| [YOUR_VAR_4]                    | [e.g. Firestore collection name]     |
| [ADD MORE AS NEEDED]            |                                      |

### Repository
- Repo URL: [e.g., https://github.com/your-org/your-repo]
- Branch to work on: [e.g., feature/kafka-listener]
- Base branch: [e.g., main]
```

---

## SECTION 4: Component-Specific Requirements

```
### What to Build
[Be specific. Include all functional requirements.]

Example:
- Listen to Kafka topic: ${KAFKA_INPUT_TOPIC}
- Deserialize each message as JSON into schema: [describe schema]
- Pass message to use case: [UseCase name]
- On success: commit Kafka offset and persist result to Firestore collection ${FIRESTORE_COLLECTION}
- On failure: publish original message to dead-letter topic ${KAFKA_DLT_TOPIC}, log error with full context

### Input Schema
[Describe the input message or request format]
Example:
{
  "order_id": "string",
  "customer_id": "string",
  "items": [{ "sku": "string", "quantity": "int" }],
  "timestamp": "ISO 8601 string"
}

### Output / Side Effects
[Describe what the component produces or does]
Example:
- Order record written to Firestore
- Confirmation event published to ${KAFKA_OUTPUT_TOPIC}

### Non-Functional Requirements
- Throughput: [e.g., up to 1000 messages/sec]
- Latency: [e.g., p99 < 200ms]
- Availability: [e.g., auto-restart on crash]

### Explicitly Out of Scope
[List what Devin should NOT build — avoids scope creep]
- [e.g., Authentication — handled by a separate component]
- [e.g., Frontend or UI]
- [e.g., CI/CD pipeline]
```

---

## SECTION 5: Definition of Done

```
Devin, your task is complete when ALL of the following are true:
1. All implementation files are created in the correct src/ directories
2. All unit tests pass with no failures
3. All integration tests pass with no failures
4. No hardcoded secrets or environment-specific values in code
5. Structured logging is in place for key operations
6. A brief README.md is added to the component folder explaining how to run it
7. A PR is opened against branch: [base branch name]
```

---

## How to Use This Template with Devin

### Option A — Paste directly into Devin's chat
1. Fill in all sections above
2. Copy the entire filled template
3. Open [app.devin.ai](https://app.devin.ai)
4. Paste into the task input box and hit run
5. Devin will plan and execute autonomously

### Option B — Store as a Playbook in Devin (Recommended for reuse)
1. In Devin, go to **Playbooks** (Teams plan)
2. Create a new Playbook and paste your filled Layer 1 + Layer 2 (Sections 1–3)
3. Save it as e.g. `standard-component-playbook`
4. For each new component, just paste Section 4 (your specific requirement) and reference the playbook:
   > "Use playbook: standard-component-playbook. New requirement: [paste Section 4 here]"
5. This way your standards and environment context are stored once — you only write the new requirement each time

### Option C — Store in your repo and point Devin to it
1. Save this filled template as `devin/task-template.md` in your repo
2. Tell Devin:
   > "Read devin/task-template.md in this repo for project standards and environment context. 
   >  Then implement the following requirement: [paste Section 4 here]"
3. Devin will read the file and use it as context automatically
