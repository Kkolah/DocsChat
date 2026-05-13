# Devin Master Template
> Store this file in your repo at: `devin/master-template.md`
> This file never changes. It defines project-wide standards for all Devin tasks.
> For each new component, attach a filled `devin-requirement-spec.md` alongside this file.

---

## AUTONOMOUS TASK INSTRUCTIONS

```
You are an expert software engineer working on a Google Cloud ADK project
using Hexagonal Architecture (Ports & Adapters).

Complete the attached requirement spec fully and autonomously:
- Do NOT ask clarifying questions
- If anything is unclear, make a reasonable assumption and document it in a code comment
- Write all implementation code
- Write unit tests and integration tests
- Run the tests — if any fail, fix the code and re-run
- Keep iterating (code → test → fix → re-test) until ALL tests pass
- Do not stop or ask for help until the full loop is complete and all tests are green
- Open a PR when all tests pass
```

---

## HOW DEVIN'S AUTONOMOUS LOOP WORKS

```
1. Read this master template + the attached requirement spec
2. Plan the implementation steps
3. Write implementation code (src/ directories)
4. Write unit tests + integration tests (tests/ directories)
5. Run the tests
   ├── All pass? → Done. Open a PR.
   └── Any fail? → Analyse failures → Fix the code → Go back to step 5
```

No human intervention needed between steps. You review the final PR only.

---

## ARCHITECTURE STANDARDS

```
### Hexagonal Architecture (Ports & Adapters)
- Core domain logic must NEVER depend on infrastructure
- Define a Port (interface) for every external system interaction
- Implement an Adapter for each external system (Kafka, DB, HTTP, GCP, etc.)
- Dependencies always point inward — domain → ports → adapters

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
```

---

## CODE STANDARDS

```
### General
- Follow SOLID principles
- Write clean, self-documenting code with meaningful names
- Keep functions small and single-purpose
- No magic numbers or hardcoded values — use constants or config

### Security
- NEVER hardcode secrets, credentials, or sensitive config
- Read all secrets from environment variables
- Validate and sanitize all external inputs
- Follow the principle of least privilege

### Error Handling
- Handle all exceptions explicitly — never swallow errors silently
- Use structured error types (custom exceptions where appropriate)
- Log errors with sufficient context for debugging
- Implement retry logic with exponential backoff for transient failures

### Logging & Observability
- Use structured JSON logging
- Log at appropriate levels: DEBUG, INFO, WARN, ERROR
- Include correlation IDs / trace context in all log entries
- Emit metrics for key operations (latency, success/failure counts)

### Configuration
- Inject all dependencies — no hidden globals
- All config comes from environment variables — nothing environment-specific in code
- Use dependency injection patterns appropriate to the language/framework

### Code Style
- Async/await throughout (if applicable to language)
- All dates in UTC ISO 8601 format
- [Add project-specific conventions here]
```

---

## GOOGLE CLOUD ADK STANDARDS

```
- Use Google Cloud ADK idioms and best practices
- Use Application Default Credentials (ADC) for GCP authentication
- Use ADK-native clients for GCP services (Pub/Sub, Firestore, BigQuery, etc.)
- Structure agents with clear input/output schemas
- Implement proper agent lifecycle hooks: init, run, shutdown
- Use ADK's built-in observability and tracing where available
```

---

## TESTING STANDARDS

```
- Unit tests for ALL domain logic and use cases
- Integration tests for ALL adapters
- Mock all external dependencies in unit tests
- Aim for high coverage on critical paths
- All tests must pass before task is considered complete
- Test runner: [e.g., pytest / mvn test / npm test]
```

---

## ENVIRONMENT & CONFIGURATION

```
### Runtime Environment
- Language: [e.g., Python 3.11 / Java 17 / Node.js 20]
- Framework: [e.g., FastAPI / Spring Boot / Express]
- Cloud Platform: Google Cloud Platform
- ADK Version: [e.g., google-cloud-adk 1.x]
- Package manager: [e.g., pip / maven / npm]

### Environment Variables
All secrets and config are injected via environment variables.
NEVER hardcode these values. Reference them as os.environ or equivalent.

| Variable Name                   | Description                          |
|---------------------------------|--------------------------------------|
| GOOGLE_CLOUD_PROJECT            | GCP Project ID                       |
| GOOGLE_APPLICATION_CREDENTIALS  | Path to service account key (if set) |
| [YOUR_VAR_1]                    | [e.g. Kafka broker URL]              |
| [YOUR_VAR_2]                    | [e.g. Kafka input topic name]        |
| [YOUR_VAR_3]                    | [e.g. Kafka DLT topic name]          |
| [YOUR_VAR_4]                    | [e.g. Firestore collection name]     |
| [ADD MORE AS NEEDED]            |                                      |

### Repository
- Repo URL: [e.g., https://github.com/your-org/your-repo]
- Base branch: [e.g., main]
```

---

## DEFINITION OF DONE

```
Task is complete ONLY when ALL of the following are true:
1. All implementation files created in correct src/ directories
2. All unit tests pass with no failures
3. All integration tests pass with no failures
4. No hardcoded secrets or environment-specific values in code
5. Structured logging in place for all key operations
6. A brief README.md added to the component folder
7. PR opened against the base branch
```
