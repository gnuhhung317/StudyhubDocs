# Local Development Setup Guide

This guide will walk you through setting up the entire StudyHub application on your local machine for development.

## Prerequisites

- **Docker and Docker Compose:** For running infrastructure services.
- **Java 21 (or higher) JDK:** For running the Spring Boot applications.
- **Maven:** For building the Java projects.
- **Git:** For version control.
- **An IDE:** IntelliJ IDEA is recommended.

## Step-by-Step Setup

1.  **Clone the Code Repository:**
    ```bash
    git clone <URL_to_your_code_repository>
    cd studyhub
    ```

2.  **Start Infrastructure Services:**
    - All backing services (databases, message brokers, etc.) are defined in `ops/docker-compose.dev.yml`.
    - To start them, run:
    ```bash
    docker-compose -f ops/docker-compose.dev.yml up -d
    ```
    - To check if all services are running, use `docker-compose -f ops/docker-compose.dev.yml ps`.

3.  **Configure Keycloak (First-time setup only):**
    - Navigate to `http://localhost:8080` in your browser.
    - Log in to the admin console (default credentials are usually `admin`/`admin`).
    - Create a new Realm named `studyhub`.
    - Create a new Client named `studyhub-backend`.
    - Create a test user within the `studyhub` realm.

4.  **Run a Service from your IDE:**
    - Open one of the service modules (e.g., `user-service`) in IntelliJ IDEA.
    - The IDE should automatically detect it as a Maven project.
    - Find the main application class (e.g., `UserServiceApplication.java`) and run it.
    - The service will start and connect to the infrastructure services running in Docker.

5.  **Verify the Setup:**
    - Follow **Task 2.3** in the [Sprint 1-2 Plan](../sprints/sprint-01-02-foundation-auth/README.md) to perform a full authentication and API request flow.
