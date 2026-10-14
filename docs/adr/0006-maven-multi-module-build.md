# 0006. Maven multi-module build

- Status: Accepted
- Date: 2026-10-14

## Context

The engine started as a standalone Maven project and CI ran only inside `engine/`. The server must depend on the engine, and both should share Java, JUnit and plugin versions.

## Decision

The repo root `pom.xml` is a parent and aggregator (`rikken:rikken-parent`). It owns the Java release, encoding, dependency management and plugin management. `engine` is the first module, and `server` is added as a module when it exists. CI runs `mvn verify` from the repo root.

## Consequences

- One command builds and tests everything, and shared versions are bumped in one place.
- The server can depend on `rikken:engine` and Maven builds the engine first.
- Spring Boot normally provides its own parent, so the server will import `spring-boot-dependencies` as a BOM instead (see [0007](0007-spring-boot-server.md)).
- Branch protection that requires a status check named after the old `engine` job must be updated to `build`.
