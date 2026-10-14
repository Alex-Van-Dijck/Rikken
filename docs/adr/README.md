# Architecture decision records

An ADR captures one significant decision: the context, what was decided and the consequences. Rules of the game live in [rules.md](../rules.md), and the roadmap lives in [plan.md](../plan.md). ADRs explain why the technical choices behind them were made.

## Conventions

- One decision per file, named `NNNN-short-title.md` with a zero-padded sequence number.
- Copy [0000-template.md](0000-template.md) to start a new record.
- Statuses: `Proposed`, `Accepted`, `Superseded by NNNN`, `Rejected`.
- Accepted records are not rewritten. To change a decision, add a new ADR and mark the old one `Superseded by NNNN`. Typos and clarifications are fine to fix in place.

## Index

| ADR                                                  | Title                                           | Status   |
| ---------------------------------------------------- | ----------------------------------------------- | -------- |
| [0001](0001-record-architecture-decisions.md)        | Record architecture decisions                   | Accepted |
| [0002](0002-authoritative-server-with-pure-engine.md) | Authoritative server with a pure Java engine    | Accepted |
| [0003](0003-store-games-as-seed-and-action-log.md)   | Store games as seed plus action log             | Accepted |
| [0004](0004-json-protocol-with-generated-types.md)   | JSON protocol with generated TypeScript types   | Accepted |
| [0005](0005-username-only-login-in-v1.md)            | Username-only login in v1                       | Accepted |
| [0006](0006-maven-multi-module-build.md)             | Maven multi-module build                        | Accepted |
| [0007](0007-spring-boot-server.md)                   | Spring Boot for the server                      | Proposed |
