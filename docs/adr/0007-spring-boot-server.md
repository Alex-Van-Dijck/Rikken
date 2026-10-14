# 0007. Spring Boot for the server

- Status: Proposed
- Date: 2026-10-14

## Context

The server handles WebSocket connections, session tokens, persistence of the seed and action log, turn timers and bot threads. It needs a mainstream JVM framework with WebSocket and Postgres support.

## Decision

Use Spring Boot for the `server` module. The implementation language, Java or Kotlin, is still to be decided. Postgres is the database, as listed in the Decisions section of [plan.md](../plan.md).

## Consequences

- Mature WebSocket, persistence and testing support, and plenty of documentation.
- A heavier runtime than a minimal framework. Hosting is estimated at about 1 GB of RAM for the JVM, which fits the planned budget.
- Choosing Kotlin affects the build (Kotlin compiler plugin) and the engine boundary, since the engine stays Java and must be called from Kotlin.
- This record should move to `Accepted` once the language is chosen and the server module is created.
