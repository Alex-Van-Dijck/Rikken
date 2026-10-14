# 0001. Record architecture decisions

- Status: Accepted
- Date: 2026-10-14

## Context

Several technical choices were made while writing the v1 plan (server authority, protocol format, identity, build layout). They currently live in the Decisions section of [plan.md](../plan.md) without their reasoning, and plan.md will change as milestones complete.

## Decision

We record significant architecture decisions as ADRs in `docs/adr/`, using the format in [0000-template.md](0000-template.md). plan.md keeps the roadmap and links to ADRs for the reasoning.

## Consequences

- The reasoning behind a choice survives plan changes and is easy to find later.
- Changing a decision means writing a new ADR, which adds a small amount of process.
