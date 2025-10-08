# API Conventions

This document outlines the conventions all REST APIs in the StudyHub project must follow.

## URI Naming

- Use plural nouns for resources (e.g., `/users`, `/messages`).
- Use kebab-case for path segments.
- The base path for all services will be `/api/v{version}` (e.g., `/api/v1`).

## HTTP Verbs

- `GET`: Retrieve a resource or a list of resources.
- `POST`: Create a new resource.
- `PUT`: Replace an existing resource entirely.
- `PATCH`: Partially update an existing resource.
- `DELETE`: Delete a resource.

## Error Handling

- All APIs must return errors compliant with **RFC-7807 (Problem+JSON)**.
- A standard error response should look like this:

```json
{
  "type": "https://studyhub.com/errors/validation-error",
  "title": "Validation Failed",
  "status": 400,
  "detail": "The request body is invalid.",
  "instance": "/api/v1/messages",
  "errors": [
    { "field": "content", "message": "must not be empty" }
  ]
}
```

## Versioning

- API versioning will be done via the URL path (e.g., `/api/v1/...`).
