# 0005. Username-only login in v1

- Status: Accepted
- Date: 2026-10-14

## Context

Players need an identity so seats, scores and reconnects belong to someone. Full accounts (email, password hashing, verification, reset flow) are a large amount of work for a v1 with 10 to 100 daily users who mostly know each other.

## Decision

In v1 a player logs in with a username only, with no password. The server issues a session token on login and uses it to identify the player on later connections. Email and password authentication, verification, reset and OAuth are listed in [expansions.md](../expansions.md).

## Consequences

- Fastest path to a playable multiplayer game, and no credential storage to secure.
- Anyone can claim any unused username and nothing prevents impersonation of an existing one, which is acceptable only while the player base is small and trusted.
- The identity model should keep a stable internal player id separate from the username, so stronger authentication can be added later without a data migration.
