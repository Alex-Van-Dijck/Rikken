# 0002. Authoritative server with a pure Java engine

- Status: Accepted
- Date: 2026-10-14

## Context

Rikken has hidden information (hands and the hidden partner) and players are untrusted, so a client cannot be allowed to decide what is legal or see other hands. The rules are also where bugs are most likely, and they need to be testable without a network or UI.

## Decision

- The server is authoritative. Clients send intents and render what they receive.
- All rules live in the `engine` module, a pure Java library with no I/O and no frameworks. It exposes `reduce(state, action)`, `legalActions(state, seat)` and `view(state, seat)`, uses a seeded RNG, and keeps state immutable.
- Each `PlayerView` includes the legal actions, so the client contains no rules code.
- Bots implement the same player interface as humans and run in the server process.
- One server instance runs one serialized queue per game.

## Consequences

- The engine can be tested heavily in isolation, including random-game property tests and golden replays.
- Hidden information never leaves the server, because `view` is the only way state reaches a client.
- Offline play would need a second engine implementation in TypeScript, kept honest with shared golden replays (see [expansions.md](../expansions.md)).
- A single server instance is a deliberate limit that fits the expected 10 to 100 daily users, and it avoids distributed game state.
