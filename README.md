# Rikken

An online trick-taking card game: base Rikken for four players, with bots filling empty seats. Turn-based play over WebSocket, authoritative Java server, React web client.

## Quick start

```bash
# Build and test the engine
mvn verify

# Run a single random game
mvn -pl engine exec:java -Dexec.mainClass="rikken.RandomGame"
```

## What is Rikken?

Rikken is a Belgian trick-taking card game for four players. One player (the bidder) and a secret partner bid to win a certain number of tricks, and the other two defend. The partner's identity is hidden until they play the called card.

Full rules are in [docs/rules.md](docs/rules.md).

## Project structure

```
engine/          Pure Java rules engine (no I/O, heavily tested)
server/          Spring Boot WebSocket server (placeholder)
client/          React web client (placeholder)
protocol/        JSON Schema for messages (placeholder)
docs/
  rules.md       Complete Rikken rules specification
  plan.md        Roadmap and architecture decisions
  adr/           Architecture decision records
```

**The engine** is the heart of the project: a deterministic, immutable rules engine that runs the game. It exports three functions: `reduce(state, action)` applies an action to state, `legalActions(state, seat)` computes the legal moves for a seat, and `view(state, seat)` returns what a seat can see (hand, public info, legal actions). Games are seeded, so the same seed plus the same sequence of actions always produces the same outcome. The engine is tested with table-driven tests and property-based tests that play thousands of random legal games.

**The server** is authoritative. Clients send intents (`Bid`, `PlayCard`, etc.) and render what the server sends back (`GameView` with legal actions). All rules live in the engine; the server orchestrates games over WebSocket and persists them as seed plus action log.

**The client** is a thin display layer. It imports generated TypeScript types from the protocol schema but never imports the engine or server. All game logic is on the server.

## Development

### Architecture decisions

Reasoning for major choices is recorded in [docs/adr/](docs/adr/). Read [0002-authoritative-server-with-pure-engine.md](docs/adr/0002-authoritative-server-with-pure-engine.md) for an overview of how the system fits together.

### Build

The project uses Maven with a multi-module layout (root parent + engine + server):

```bash
mvn verify                      # Build and test everything
mvn -pl engine verify           # Build and test engine only
mvn -pl engine test -Dtest=RandomGameTest  # Run a specific test
```

CI runs on every push to main. See [.github/workflows/main.yml](.github/workflows/main.yml).

### Testing strategy

- **Unit tests**: table-driven tests for individual functions (trick winner, legal plays, scoring)
- **Property tests**: random legal games (10,000 games per run, checking invariants like card conservation and score consistency)
- **Golden replays**: seed + action log from a known game can be replayed to the same end state
- **Integration tests**: added when the server vertical slice exists

## Technology

- **Engine**: Java 21, JUnit Jupiter 6.1.3
- **Server**: Spring Boot (Java or Kotlin, TBD)
- **Client**: React
- **Protocol**: JSON over WebSocket
- **Database**: Postgres
- **Hosting**: Fly.io or Hetzner, ~7–8 USD per month

## Design principles

- **Server is authoritative.** The engine is pure Java with no I/O. Clients are thin display layers that send intents and render what the server sends.
- **Rules live in one place.** All game logic is in the engine, testable in isolation.
- **State is immutable.** Every action returns a new state; no mutation.
- **Games are deterministic.** Seeded RNG means the same seed and action log always produces the same game.
- **Hidden information stays hidden.** `view(state, seat)` is the only way state reaches a client, so the hidden partner and other hands never leak.

## Scope for v1

- Online play only (web app, mobile comes later)
- Base Rikken, four players only
- 10 to 100 daily users, no spectators, no push notifications
- Bots fill empty seats
- Turn timers handle absent players

Out of scope for v1 (see [docs/expansions.md](docs/expansions.md)):

- Offline play
- Spectators
- Stronger bots (ISMCTS)
- Game variants (solo modes, pass games)
- Email and password authentication (username-only in v1)
- Push notifications

## Contributing

(This is a solo project for now, but if you're reading this you're probably collaborating with the author.)

See [CLAUDE.md](.claude/CLAUDE.md) for conventions on comments, naming and emoji use.
