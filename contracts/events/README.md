# Event Contracts

This directory contains the schemas for events published to our messaging systems (primarily Kafka).

We use **JSON Schema** to define the structure and data types of our events.

## Naming Convention

- **Topic Name:** `domain.resource.action` (e.g., `chat.messages.created`).
- **Schema File Name:** `{eventName}-v{version}.json` (e.g., `chat-message-created-v1.json`).

## Schemas

- **[ChatMessageCreated (v1)](./chat-message-created-v1.json):** Published when a new chat message is successfully persisted.
