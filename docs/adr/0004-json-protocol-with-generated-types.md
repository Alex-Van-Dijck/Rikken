# 0004. JSON protocol with generated TypeScript types

- Status: Accepted
- Date: 2026-10-14

## Context

The Java server and the TypeScript client must agree on every message (Bid, PlayCard, GameView and so on). Maintaining the types by hand on both sides drifts over time.

## Decision

- Messages are JSON, sent over WebSocket.
- The `protocol` directory holds a JSON Schema describing every message. TypeScript types for the client are generated from it.
- The client imports generated types only and never imports the engine or the server.

## Consequences

- JSON is easy to inspect and debug, at the cost of larger messages than a binary format. Message size is irrelevant for a turn-based game.
- The schema is the contract, so a message change starts in the schema.
- Open question: which side is the source of truth for the schema, and how the Java side is validated against it. Decide when Milestone 3 starts.
