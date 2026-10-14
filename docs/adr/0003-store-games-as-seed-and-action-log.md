# 0003. Store games as seed plus action log

- Status: Accepted
- Date: 2026-10-14

## Context

The server must persist games, resume them after a restart, and let a reconnecting client resync. The engine is deterministic given a seed and the sequence of actions.

## Decision

Every game is persisted as its seed plus an ordered log of actions. Current state is rebuilt by replaying the log through the engine.

## Consequences

- Storage is small, and any game can be replayed for debugging or turned into a golden test.
- Replay only reproduces a game if the engine behaves the same, so each stored game should record the engine or rules version. This needs to be added when persistence is built.
- Actions taken by the system, such as a turn timeout or a bot taking over a seat, must be written to the log like any other action, or replay will diverge.
- Rebuilding state costs a replay per load, which is negligible at this scale.
