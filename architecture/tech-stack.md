# Tech Stack

This document lists the official technologies, frameworks, and libraries for the StudyHub project.

## Backend
- **Framework:** Spring Boot 3.x
- **Language:** Java 21
- **API Gateway:** Spring Cloud Gateway
- **Authentication:** Spring Security (OAuth2 Resource Server), Keycloak
- **Database:** PostgreSQL
- **ORM & Migration:** Spring Data JPA, Flyway
- **Messaging:** Apache Kafka (Events), RabbitMQ (Tasks)
- **Caching:** Redis
- **Search:** Elasticsearch
- **Resilience:** Resilience4j
- **Testing:** JUnit 5, AssertJ, Testcontainers, WireMock, Pact
- **Observability:** Micrometer, OpenTelemetry (Jaeger/Zipkin), Prometheus, Grafana
- **Build Tool:** Maven

## Infrastructure
- **Containerization:** Docker
- **Orchestration:** Kubernetes (via Helm)
- **CI/CD:** GitHub Actions
- **Secrets Management:** HashiCorp Vault (or Kubernetes Secrets)
- **File Storage:** MinIO (S3-compatible)

## Frontend (Assumed)
- **Framework:** React / Next.js or Vue.js
- **Styling:** Tailwind CSS or Material-UI
- **State Management:** Redux or Pinia
