# Online trick-taking card game: analysis and plan

## Scope of version 1

- Online play only, starting with a React web app
- Java backend with an authoritative server
- 10 to 100 daily users, no spectators, no push notifications
- Base Rikken for four players, with bots filling empty seats
- Out of scope for now: offline play, spectators, push notifications, stronger (ISMCTS) bots, game variants and pass games (passpellen)
- Mobile (Expo / React Native) comes after the web app is stable

## Analysis

### What makes this easier than it looks

- Turn-based, so latency and bandwidth barely matter
- Tiny load: a few dozen concurrent sockets at peak, one small server is enough
- All rules live in one pure Java engine, so the hardest logic is testable without a UI or a network
- Hosting is roughly 7 to 8 dollars per month (about 1 GB of RAM for the JVM), plus app store fees

### What will take the most care

- Turning the attached Rikken rules into an exact spec before coding: bid order and values, the called-ace partner, follow-suit rules, scoring
- Reconnects: phones suspend apps and drop sockets, so the client must resync from the server's current view
- Abandoned games: without push notifications you need turn timers and a bot that takes over an absent seat
- Keeping the wire protocol in sync between Java and TypeScript: define it in a schema and generate the TS types
- Identity: players log in, so pick the sign-in method early, because it shapes the database and the protocol
- Matchmaking: all tables are public, so you need a lobby list, seat claiming and a way to handle players who leave mid-game

## Architecture decisions

- Server is authoritative: the client sends intents and renders what it receives
- The engine is a pure Java library: reduce(state, action), legalActions, view. No I/O, seeded RNG
- Every game is stored as seed plus action log
- The client has no rules code: each PlayerView includes the legal actions
- Bots implement the same Player interface as humans, run in-process and use their own thread pool
- One server instance, one serialized queue per game
- Database: Postgres (free managed tier or self-hosted)
- Hosting: a small Fly.io machine or a Hetzner VPS

## Repository structure

- **/engine**: Pure Java library with game rules. Exports reduce(state, action), legalActions(state, seat), view(state, seat) and everything needed to run a game. No I/O, no frameworks. This is where bugs hide and where tests catch them, so it lives separately and is heavily tested.
- **/server**: Spring Boot runtime (Java or Kotlin, to be decided) that orchestrates games over WebSocket. Manages player sessions, seat claiming, turn timers, bot threads, persistence (seed + action log), and authentication. Uses /engine to drive game logic.
- **/protocol**: JSON Schema that defines all JSON messages between server and client (Bid, PlayCard, GameView, etc.). Generates TypeScript types so both sides stay in sync without manually maintaining types.
- **/client**: React web client. Sends actions as intents, receives GameView + legal actions, renders the table. Imports generated types from /protocol but never imports /engine or /server.

This split keeps concerns separate: the engine is tested in isolation, the server orchestrates without game logic, the protocol is the contract, and the client is a thin display layer.

## Plan

### Milestone 0: rules and repository (small)

- Write a one-page rules spec for base Rikken ([rules.md](rules.md)), starting from the attached nl-BE rules and resolving every (assumption) and (open) marker in it
- Create the repo with the four directories above, plus CI that builds and tests the Java side
- Done when: the spec is reviewed and CI is green on an empty build

### Milestone 1: engine (medium)

- Types: Card, Seat, GameState, Action and GameEvent as records and sealed interfaces
- Seeded dealer, legal plays, trick winner, bidding, scoring, reduce, view
- Tests: table tests for the rules, property-based tests that play thousands of random legal games, golden replays
- A game is a series of deals that ends when a player reaches 50 points (rules.md section 6)
- Done when: 10,000 random games finish with all invariants holding (cards conserved, scores consistent, every game terminates)

### Milestone 2: heuristic bots and headless games (small)

- Player interface, a random bot and a simple heuristic bot
- A headless runner that plays full bot-only games
- Done when: bots play complete games with no illegal moves and the heuristic bot clearly beats the random bot

### Milestone 3: server vertical slice (medium)

- WebSocket endpoint, protocol messages defined in a schema, one orchestrator per game
- Persist seed and action log, and load a game back from its log
- Login by username only (no password in v1), with a session token issued on login
- Done when: a scripted test client plays a full game against three bots over WebSocket and the game replays from the database

### Milestone 4: web client vertical slice with React (medium)

- Generate TS types from the schema; keep the connection layer (with reconnect), store and event queue in plain TypeScript modules with no React imports
- Table screen built from React components, desktop-first, with hand area, trick area, bidding, scores, and playable cards from legal actions
- Event queue that drives animations and small bot delays
- Done when: you can play a full game against three bots in a web browser

### Milestone 5: real multiplayer (medium)

- Public lobby: list open tables, create and join a table, seats filled by humans or bots
- Resync after reconnect, turn timers, bot takeover for absent players
- Done when: four people can finish a game, including one who drops and rejoins

### Milestone 6: deploy and beta (small)

- Deploy to Fly.io or Hetzner, add logging, basic monitoring and database backups
- Beta test with friends for a week without lost games
- Done when: web app is stable and ready for wider testing

### Milestone 7: mobile port (medium, post-v1)

- Adapt React app to React Native / Expo using react-native-web for code sharing
- Portrait-first mobile layout
- First phone builds, internal test builds, then store accounts and beta testers
- Done when: app runs on iOS and Android with the same game logic

### Later

- ISMCTS bots, push notifications, offline play (a TypeScript port tested against shared golden replays)

## First steps

1. Write the rules spec for base Rikken from the attached rules
2. Create the repo and the Java build with CI
3. Define Card, Seat, Action and GameEvent
4. Implement trickWinner and legalPlays with table tests, since this is where bugs hide
5. Implement the seeded dealer and a first random-game simulation

## Rules spec

The complete Rikken rules specification is in [rules.md](rules.md). It is the single source of truth for bidding, play and scoring, so this plan does not restate them.

## Decisions

Reasoning for the technical choices is recorded in [adr/](adr/README.md).

- **Game**: base Rikken for four players as specified in rules.md, without variants or pass games (passpellen)
- **Identity**: players log in with a username only in v1. Email and password auth is listed in [expansions.md](expansions.md)
- **Protocol**: JSON messages over WebSocket
- **Server**: Spring Boot, with Java or Kotlin still to be decided
- **Lobbies**: all public
- **Database**: Postgres
