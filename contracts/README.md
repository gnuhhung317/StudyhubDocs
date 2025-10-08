# Service Contracts

This section defines the contracts that govern communication between our microservices. Adhering to these contracts is critical for maintaining a stable, decoupled architecture.

## Principles

1.  **Contracts are Law:** Once a contract version is published and in use, it should be treated as immutable. Breaking changes require a new version.
2.  **Consumer-Driven:** Contracts should be designed with the consumer's needs in mind.
3.  **Explicit Versioning:** All contracts (API and event) must be versioned.

## Contract Types

- **[API Contracts (OpenAPI)](./openapi/README.md):** For synchronous request/response communication (REST APIs).
- **[Event Contracts (JSON Schema)](./events/README.md):** For asynchronous event-based communication (Kafka).
- **[API Conventions](./api-conventions.md):** General rules that apply to all APIs (e.g., error handling, naming).
