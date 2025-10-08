# How to Create a Spec for the StudyHub Project

This guide provides the commands to initiate the Spec-Driven Development (SDD) process for the StudyHub project, based on the principles outlined in the main project guide.

Follow these steps using an AI agent that supports the SDD commands (e.g., Gemini CLI).

## 1. Initialize the Spec Environment

First, initialize the spec-driven development environment within the project directory. This command sets up the necessary scripts and templates.

```bash
# Run this from the root of the `learning-project` directory
specify init --here --ai gemini
```

## 2. Define the Project Constitution

Use the `/constitution` command to set the governing principles for the project. This is based on the architectural vision from the project guide.

```bash
/constitution Create principles for the StudyHub project. The architecture is Microservices-based, using "Database per service". Communication is synchronous (REST/gRPC) via an API Gateway and asynchronous (Kafka for domain events, RabbitMQ for tasks). The tech stack is Spring Boot 3 on Java 21. Infrastructure is containerized with Docker and orchestrated with Kubernetes. The system must be highly observable (Prometheus, Grafana, Jaeger) and secure (OAuth2/OIDC with Keycloak). Development must follow Test-Driven Development using Testcontainers and define clear API/event contracts (OpenAPI, Pact).
```

## 3. Create the Project Specification

Use the `/specify` command to generate the functional specification. The prompt should describe the *what* and *why* of the project.

```bash
/specify Build a real-time collaborative learning platform called StudyHub. Key features include: 1. **Communication:** Real-time chat, voice/video calls within group study rooms, including screen sharing and a collaborative whiteboard. 2. **Content Streaming:** Users can upload videos, which are processed for streaming, and watch live-streamed lectures. 3. **AI Assistant:** An integrated AI chatbot that can answer questions about uploaded documents, summarize video lectures via speech-to-text, and generate quizzes. It should also support Text-to-Speech for accessibility. 4. **Universal Search:** A powerful search engine to find content across chat logs, document text, and video transcripts. 5. **Real-time Collaboration:** Users can edit documents together in real-time. 6. **Enhanced Experience:** The application should be a Progressive Web App (PWA) for offline use and support web push notifications. 7. **Security:** Users can log in via social providers like Google/LinkedIn and use modern standards like Passkeys. The architecture must be microservices-based, broken down into the following bounded contexts: auth-service, user-service, chat-service, realtime-service, media-service, ai-service, and search-service.
```

## 4. Create the Technical Implementation Plan

Use the `/plan` command to provide the specific technology stack and architectural choices. This defines *how* the project will be built.

```bash
/plan The application will be built using the Java ecosystem. The core framework is Spring Boot 3.x with Java 21. Use Spring Cloud Gateway for the API Gateway. For data persistence, use Spring Data JPA with PostgreSQL and Flyway for database migrations. Messaging will be handled by spring-kafka for domain events and spring-amqp (RabbitMQ) for background tasks. Use Redis for caching, Elasticsearch for search capabilities, and an S3-compatible object store like MinIO for file storage. Secure the services using Spring Security's OAuth2 Resource Server capabilities, with Keycloak as the Identity Provider. For testing, the stack includes JUnit5, AssertJ, Testcontainers for integration tests, and Pact for contract testing.
```

## 5. Generate and Review Tasks

Once the plan is generated and reviewed, use the `/tasks` command to break it down into an actionable list.

```bash
/tasks
```

## 6. Execute Implementation

After reviewing the generated tasks, use the `/implement` command to have the AI agent build the application.

```bash
/implement
```
