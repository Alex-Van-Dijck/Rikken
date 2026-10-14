# Rikken expansions: out-of-scope ideas

Ideas and variants to implement after v1 ships. These are NOT in scope for the initial release but are designed to be added incrementally.

## Game modes

### Solo modes (3v1)

- **8 tricks alone** (acht alleen): solo player bids to win 8 tricks against the other three
- **Misère** (no tricks alone): solo player bids to lose all tricks

### Troela mode

Solo mode variant where the bidder picks trump and leads the called ace immediately. Rules and trick thresholds TBD.

### Everyone passes variant

Alternative rule when all four players pass: determine who plays the next round (e.g., random, dealer stays, next player rotates) instead of automatic redeal.

## Authentication

Current v1 uses simple username login. Future improvements:

- Email and password authentication with proper hashing
- Email verification for new accounts
- Password reset flow
- OAuth integration (Google, GitHub, etc.)
- Account recovery options

## Offline play

- TypeScript port of the rules engine (tested against shared golden replays with Java version)
- Local bot play in the app
- Sync to server when connection returns
- Support for playing without internet access

## UI/UX enhancements

- Spectator mode to watch ongoing games
- Game replay system and historical stats
- Player profiles and win/loss tracking
- Elo or ranking system
- Customizable avatars and player names

## Bot improvements

- ISMCTS (Information Set Monte Carlo Tree Search) bots for stronger play
- Difficulty levels
- Bot personality variants

## Infrastructure

- Push notifications for turn alerts and game invites
- Persistent lobbies and friend lists
- Automated tournament support
- Admin moderation tools
