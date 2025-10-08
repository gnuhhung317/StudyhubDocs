# Testing Strategy

Our testing strategy is based on a multi-layered approach to ensure code quality, service correctness, and system reliability.

## 1. Unit Tests

- **Purpose:** To test a single class or component in isolation.
- **Scope:** A single Java class.
- **Tools:** JUnit 5, AssertJ, Mockito.
- **Location:** `src/test/java` in each service.
- **When to write:** Continuously, as you develop new logic.

## 2. Integration Tests

- **Purpose:** To test a service's interaction with external infrastructure like databases, message brokers, or other APIs.
- **Scope:** A single service and its infrastructure dependencies.
- **Tools:** Spring Boot's `@SpringBootTest`, **Testcontainers**.
- **Key Idea:** With Testcontainers, we spin up real Docker containers (e.g., a real PostgreSQL database, a real Kafka broker) for each test run. This provides much higher fidelity than using in-memory fakes.
- **Example:** Testing if a `POST /messages` API call correctly persists a record to a real PostgreSQL database and publishes an event to a real Kafka topic.
- **Location:** `src/test/java` in each service, typically in a separate `integration` package.

## 3. Contract Tests

- **Purpose:** To verify that a service (the "Provider") adheres to the API contract expected by its client (the "Consumer"). This prevents breaking changes.
- **Scope:** The interface between two services.
- **Tools:** **Pact**.
- **Flow:**
    1.  The Consumer test suite defines the requests it will send and the responses it expects, generating a `pact` file (the contract).
    2.  The Provider test suite then runs this `pact` file against itself to verify it honors the contract.

## 4. End-to-End (E2E) Tests

- **Purpose:** To test a complete user flow through the entire deployed system.
- **Scope:** The whole application, from the API Gateway to multiple downstream services.
- **Tools:** Postman (Newman), Cypress, or custom scripts.
- **Example:** A script that logs a user in, creates a chat message, and then verifies that another user can retrieve that message.
- **When to run:** Primarily in CI/CD pipelines against a staging environment.
