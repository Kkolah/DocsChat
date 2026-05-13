# Agentic Code Generation Prompt Template
> Universal, reusable prompt template for generating production-ready code with minimal human interaction.
> Structured in three layers: Universal Best Practices → Environment Context → Component-Specific Requirements.

---

## LAYER 1: Universal System Prompt (Static — reuse as-is)

```
You are an expert software engineer. Generate production-ready code following these principles:

### Architecture
- Apply Hexagonal Architecture (Ports & Adapters):
  - Core domain logic must be isolated from infrastructure concerns
  - Define clear Ports (interfaces) for all external interactions
  - Implement Adapters for each external system (Kafka, DB, APIs, etc.)
  - Dependencies point inward — domain never depends on infrastructure

### Code Quality
- Follow SOLID principles
- Write clean, self-documenting code with meaningful names
- Keep functions small and single-purpose
- Avoid magic numbers or hardcoded values — use constants or config

### Error Handling
- Handle all exceptions explicitly — never swallow errors silently
- Use structured error types (custom exceptions where appropriate)
- Log errors with sufficient context for debugging
- Implement retry logic with exponential backoff for transient failures

### Logging & Observability
- Use structured logging (JSON format preferred)
- Log at appropriate levels: DEBUG, INFO, WARN, ERROR
- Include correlation IDs / trace context in log entries
- Emit metrics for key operations (latency, success/failure counts)

### Security
- Never hardcode secrets, credentials, or sensitive config
- Read all secrets from environment variables or a secrets manager
- Validate and sanitize all external inputs
- Follow the principle of least privilege

### Testing
- Write unit tests for all domain/business logic
- Write integration tests for adapters
- Aim for high coverage on critical paths
- Use mocks/stubs for external dependencies in unit tests

### Dependencies & Configuration
- Inject all dependencies (no hidden globals)
- Externalize all configuration — nothing environment-specific in code
- Use dependency injection patterns appropriate to the language/framework

### Google Cloud ADK Specifics
- Use Google Cloud ADK idioms and best practices
- Prefer ADK-native clients for GCP services (Pub/Sub, Firestore, BigQuery, etc.)
- Use Application Default Credentials (ADC) for authentication
- Structure agents with clear input/output schemas
- Implement proper agent lifecycle hooks (init, run, shutdown)
- Use ADK's built-in observability and tracing where available
```

---

## LAYER 2: Environment Context (Fill in per project)

```
### Runtime Environment
- Language: [e.g., Python 3.11 / Java 17 / Node.js 20]
- Framework: [e.g., FastAPI / Spring Boot / Express]
- Cloud Platform: Google Cloud Platform
- ADK Version: [e.g., google-cloud-adk 1.x]

### Configuration & Secrets
All sensitive values are injected via environment variables. Reference them as follows:

| Variable Name              | Description                        |
|----------------------------|------------------------------------|
| [ENV_VAR_1]                | [What it is, e.g. Kafka broker URL]|
| [ENV_VAR_2]                | [e.g. Kafka topic name]            |
| [ENV_VAR_3]                | [e.g. Database connection string]  |
| [ENV_VAR_4]                | [e.g. API key for downstream service] |
| GOOGLE_CLOUD_PROJECT       | GCP project ID                     |
| GOOGLE_APPLICATION_CREDENTIALS | Path to service account key (if not using ADC) |

### Project Structure Convention
Follow this hexagonal layout:
src/
  domain/         # Core business logic, entities, value objects
  ports/          # Interfaces (inbound + outbound)
  adapters/       # Implementations of ports (Kafka, DB, HTTP, etc.)
  application/    # Use cases / application services
  config/         # Configuration loading
  main.py         # Entrypoint (or Main.java, index.ts, etc.)

### Additional Project Conventions
- [e.g., use async/await throughout]
- [e.g., all dates in UTC ISO 8601]
- [e.g., snake_case for variables, PascalCase for classes]
```

---

## LAYER 3: Component-Specific Requirement (Change per task)

```
### Component to Build
[Describe what you need, e.g.:]
Build a Kafka consumer component that:
- Listens to topic: ${KAFKA_TOPIC}
- Connects using bootstrap servers: ${KAFKA_BOOTSTRAP_SERVERS}
- Authenticates with SASL/SSL using credentials from environment
- Deserializes messages as JSON
- Passes each message to the domain use case: ProcessOrderUseCase
- Commits offsets only after successful processing
- On failure, sends the message to a dead-letter topic: ${KAFKA_DLT_TOPIC}

### Inputs / Outputs
- Input: [describe message schema or request shape]
- Output: [describe side effects, return values, or events emitted]

### Non-Functional Requirements
- Throughput: [e.g., handle up to 1000 messages/sec]
- Latency: [e.g., p99 < 200ms]
- Availability: [e.g., auto-restart on failure]

### Do NOT include
- [e.g., authentication logic — that is handled by a separate component]
- [e.g., UI or frontend code]
```

---

## How to Use This Template

1. **Copy Layer 1** into the `system` prompt of your API call — never change it.
2. **Fill in Layer 2** once per project and store it in your repo (e.g., `prompts/env-context.md`).
3. **Write Layer 3** for each new component — this is your only per-task input.
4. **Combine all three layers** into a single prompt and send to the model:

```python
system_prompt = LAYER_1  # static

user_prompt = f"""
{LAYER_2}  # project environment context

{LAYER_3}  # this component's specific requirement
"""
```

5. The model generates complete, production-ready code with no further prompting needed.

---

## Fully Agentic Workflow (Zero Human Interaction)

For true agentic, zero-interaction code generation — including writing tests, running them, and fixing errors automatically — use **Claude Code** (Anthropic's CLI agent).

### Setup: CLAUDE.md (Project Memory File)

Create a `CLAUDE.md` file at the root of your repo. Claude Code reads this automatically on every run — it's your persistent project context, so you never repeat yourself.

```markdown
# CLAUDE.md — Project Instructions for Claude Code

## Architecture
- Hexagonal Architecture (Ports & Adapters) — see src/ layout below
- Domain logic never depends on infrastructure
- All external systems accessed only through Ports

## Tech Stack
- Language: [Python 3.11 / Java 17 / etc.]
- Framework: [FastAPI / Spring Boot / etc.]
- Platform: Google Cloud Platform, Google ADK [version]

## Project Structure
src/
  domain/       # Entities, value objects, domain logic
  ports/        # Inbound + outbound interfaces
  adapters/     # Kafka, DB, HTTP, GCP implementations
  application/  # Use cases
  config/       # Config loading from env vars
  main.py       # Entrypoint

## Environment Variables
All secrets come from environment. Never hardcode. Key vars:
- KAFKA_BOOTSTRAP_SERVERS, KAFKA_TOPIC, KAFKA_DLT_TOPIC
- GOOGLE_CLOUD_PROJECT, GOOGLE_APPLICATION_CREDENTIALS
- [Add your project-specific vars here]

## Code Standards
- SOLID principles, clean code, meaningful names
- Structured JSON logging with correlation IDs
- Retry with exponential backoff for transient failures
- Unit tests for domain logic, integration tests for adapters
- All dates UTC ISO 8601, async/await throughout

## Testing
- Run tests with: [pytest / mvn test / npm test]
- Fix all test failures before considering a task complete
- Mock all external dependencies in unit tests
```

### How to Run Agentically

```bash
# Install Claude Code
npm install -g @anthropic-ai/claude-code

# One-shot command — Claude reads CLAUDE.md, your spec, writes code + tests, runs them, fixes errors
claude "Read requirements/kafka-listener-spec.md and implement the component fully. 
Write unit and integration tests. Run the tests and fix any failures. 
Do not ask for confirmation — complete the full task autonomously."
```

### Recommended File Layout

```
your-project/
  CLAUDE.md                        # ← Persistent project context (Layer 1 + 2)
  requirements/
    kafka-listener-spec.md         # ← Your component spec (Layer 3)
    database-adapter-spec.md       # ← Another component spec
  src/                             # ← Claude writes code here
  tests/                           # ← Claude writes tests here
```

### Workflow Summary

| Step | What you do | What Claude Code does |
|------|-------------|----------------------|
| Once | Create `CLAUDE.md` with project context | Reads it automatically every run |
| Per component | Write a spec file in `requirements/` | Plans, codes, tests, fixes — autonomously |
| Never | Interact mid-task | Completes the full loop without prompting |

---

## Example: Kafka Listener (Filled-in Layer 3)

```
### Component to Build
Build a Kafka consumer that:
- Listens to topic: ${KAFKA_INPUT_TOPIC}
- Bootstrap servers: ${KAFKA_BOOTSTRAP_SERVERS}
- SASL username: ${KAFKA_SASL_USERNAME}
- SASL password: ${KAFKA_SASL_PASSWORD}
- Deserializes messages as JSON into an OrderEvent schema
- Calls ProcessOrderUseCase with each event
- On success: commit offset
- On failure: publish to ${KAFKA_DLT_TOPIC} and log error with full context

### Inputs
OrderEvent { order_id: str, customer_id: str, items: List[Item], timestamp: str }

### Outputs
Side effect: order persisted to Firestore via OutboundOrderPort
```
