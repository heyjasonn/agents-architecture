---
name: implement-go-handler
description: Implement and refactor Go HTTP/gRPC handlers that follow clean transport-layer conventions. Use when a user asks to add or update handlers, enforce handler-service-repository separation, add request validation, map request DTO to domain input, map domain errors to transport errors, keep response schema consistent, and ensure tracing/context plus structured logging in handler paths.
---

# Implement Go Handler

Implement transport handlers in Go without leaking business logic into the handler layer.

## Workflow

1. Locate the transport entrypoint and framework conventions:
- HTTP router/controller style and response envelope.
- gRPC server interface and proto-generated contracts.
- existing error mapping and logging/tracing helpers.
2. Define handler input contract:
- Parse path/query/header/body (HTTP) or request message (gRPC).
- Validate required fields, format, and bounds at transport layer.
- Return validation errors early with framework-consistent status/code.
3. Map transport request to internal DTO/domain input:
- Build an explicit input struct for service call.
- Normalize types and defaults in mapping step only.
4. Call service layer exactly once per operation intent:
- Pass `context.Context` through unchanged.
- Do not embed domain decisions, branching rules, or persistence logic in handler.
5. Map service/domain errors to transport errors:
- HTTP: map to correct status code + stable error body.
- gRPC: map to proper `codes.Code` + details when convention supports it.
- Keep unknown errors sanitized and log with structured context.
6. Map service output to response schema:
- Return field names and shape consistent with existing API/proto.
- Do not expose internal/domain-only fields unless explicitly required.
7. Add observability hooks in handler path:
- Structured logs with request identifiers and operation name.
- Trace/context propagation to service call.

## Transport Conventions

### HTTP Handler Rules

- Validate request before service invocation.
- Map errors deterministically:
- `invalid argument` -> `400 Bad Request`
- `not found` -> `404 Not Found`
- `conflict/already exists` -> `409 Conflict`
- `unauthorized` -> `401 Unauthorized`
- `forbidden` -> `403 Forbidden`
- unexpected internal errors -> `500 Internal Server Error`
- Keep response schema consistent with project response envelope.

### gRPC Handler Rules

- Validate request message fields before service invocation.
- Map errors deterministically:
- `invalid argument` -> `codes.InvalidArgument`
- `not found` -> `codes.NotFound`
- `already exists` -> `codes.AlreadyExists`
- `failed precondition` -> `codes.FailedPrecondition`
- `permission denied` -> `codes.PermissionDenied`
- `unauthenticated` -> `codes.Unauthenticated`
- unexpected internal errors -> `codes.Internal`
- Return response message matching proto schema exactly.

## Implementation Checklist

Always verify all items before finalizing:

- `no business logic in handler`
- `structured logging`
- `request validation`
- `correct status code`
- `tracing/context propagated`

## Guardrails

- Keep handler thin: parse, validate, map, call service, map response/error.
- Do not call repository from handler directly.
- Do not duplicate business rules already owned by service/domain layer.
- Keep error mapping centralized or consistent with existing helper patterns.
- Preserve backward compatibility of response contracts unless requirement says otherwise.
