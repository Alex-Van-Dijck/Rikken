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
- Reconnects: phones suspend apps and drop sockets, so the app must resync from the server's current view
- Abandoned games: without push notifications you need turn timers and a bot that takes over an absent seat
- Keeping the wire protocol in sync between Java and TypeScript: define it in a schema and generate the TS types
- Identity: players log in, so pick the sign-in method early, because it shapes the database and the protocol
- Matchmaking: all tables are public, so you need a lobby list, seat claiming and a way to handle players who leave mid-game

## Architecture decisions

- Server is authoritative: the app sends intents and renders what it receives
- The engine is a pure Java library: reduce(state, action), legalActions, view. No I/O, seeded RNG
- Every game is stored as seed plus action log
- The app has no rules code: each PlayerView includes the legal actions
- Bots implement the same Player interface as humans, run in-process and use their own thread pool
- One server instance, one serialized queue per game
- Database: Postgres (free managed tier or self-hosted)
- Hosting: a small Fly.io machine or a Hetzner VPS

## Repository structure

- **/engine**: Pure Java library with game rules. Exports reduce(state, action), legalActions(state, seat), view(state, seat) and everything needed to run a game. No I/O, no frameworks. This is where bugs hide and where tests catch them, so it lives separately and is heavily tested.
- **/server**: Java/Kotlin runtime that orchestrates games over WebSocket. Manages player sessions, seat claiming, turn timers, bot threads, persistence (seed + action log), and authentication. Uses /engine to drive game logic.
- **/protocol**: Schema that defines all messages between server and client (Bid, PlayCard, GameView, etc.). Generates TypeScript types so both sides stay in sync without manually maintaining types.
- **/app**: React web client. Sends actions as intents, receives GameView + legal actions, renders the table. Imports generated types from /protocol but never imports /engine or /server.

This split keeps concerns separate: the engine is tested in isolation, the server orchestrates without game logic, the protocol is the contract, and the app is a thin display layer.

## Plan

### Milestone 0: rules and repository (small)

- Write a one-page rules spec for base Rikken, starting from the attached nl-BE rules and settling the questions at the end of this doc
- Create the repo with the four directories above, plus CI that builds and tests the Java side
- Done when: the spec is reviewed and CI is green on an empty build

### Milestone 1: engine (medium)

- Types: Card, Seat, GameState, Action and GameEvent as records and sealed interfaces
- Seeded dealer, legal plays, trick winner, bidding, scoring, reduce, view
- Tests: table tests for the rules, property-based tests that play thousands of random legal games, golden replays
- Done when: 10,000 random games finish with all invariants holding (cards conserved, scores consistent)

### Milestone 2: heuristic bots and headless games (small)

- Player interface, a random bot and a simple heuristic bot
- A headless runner that plays full bot-only games
- Done when: bots play complete games with no illegal moves and the heuristic bot clearly beats the random bot

### Milestone 3: server vertical slice (medium)

- WebSocket endpoint, protocol messages defined in a schema, one orchestrator per game
- Persist seed and action log, and load a game back from its log
- Login (accounts, JWT sessions)
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

The complete Rikken rules specification is in [rules.md](rules.md). Key points:

- **Setup**: 4 players, 52 cards, 13 per player, seeded deal with rotating dealer
- **Bidding**: Pass < Rik 8 < Rik 9 < ... < Rik 13. Player left of dealer leads bidding; lowest bid wins contract and calls trump suit and ace. Partner is hidden until called ace plays.
- **Play**: Follow suit if able, highest trump or led suit wins trick. 13 tricks total.
- **Scoring**: Contract team wins if they take at least the bid tricks. Rik 8 pays 10 + 5 per trick, higher rik bids pay 10 + 5 per trick above 8. Non-contract team gains/loses same.

## Decisions

- **Game**: base Rikken for four players as specified in rules.md, without variants or pass games (passpellen)
- **Identity**: players sign up and log in with email and password, using JWT auth
- **Lobbies**: all public
- **Database**: Postgres
