# Rikken (base game): rules spec v1

Based on the nl-BE rules in `spelregels-rikken.pdf` [1] and the v1 decisions. Items marked **(assumption)** or **(open)** are not in the source or the decisions and need confirmation before coding.

## 1. Setup

- 4 players. Bots fill empty seats.
- 52 cards, no jokers. Rank runs from 2 (low) to ace (high) [1].
- 13 cards per player **(assumption)**.
- Seeded deal. The dealer rotates clockwise **(assumption)**.

## 2. Bidding

- The player left of the dealer bids first, then play continues clockwise. A player who has passed cannot bid again [1].
- Bids, lowest to highest: `Pass` < `Rik 8` < `Rik 9` < … < `Rik 13`.
  - A rik bid is the number of tricks the bidder and partner must win together.
  - Each bid must be higher than the previous one.
- Out of scope: _betere rik_, _acht alleen_, _piek/piccolo_, _misère_, _troela_ and pass games. Every game is 2 vs 2.
- If everyone passes, the cards are redealt by the next dealer.

## 3. Contract set-up

### Rik

- After the others have passed, the highest bidder names the trump suit and the ace they call. The holder of that ace is the partner [1].
- The partner may not reveal themselves. It becomes clear when the called ace is played [1].
- The bidder may not call an ace they hold themselve.

## 4. Play

- The player left of the dealer leads the first trick. After that, the trick winner leads.
- Players must follow suit if they can. Otherwise they may play any card, including a trump.
- The highest trump wins the trick. If no trump was played, the highest card of the led suit wins.
- 13 tricks are played. Tricks are counted per team.

## 5. Result and scoring

- The contract team wins if it takes at least the bid number of tricks.

| Contract | Win                                                | Loss                                    |
| -------- | -------------------------------------------------- | --------------------------------------- |
| Rik 8    | 10 + 5 per overtrick [1]                           | 10 + 5 per trick short (minimum 15) [1] |
| Rik 9–13 | 10 + 5 per trick above 8, measured against the bid | 10 + 5 per trick short from bid         |

- The non-contract team gains / loses as many points.

## 6. Engine notes

- **State:** seed, dealer, bids, contract, trump (nullable until revealed), hidden partner, tricks, hands.
- **Actions:** `Bid(n)`, `Pass`, `ChooseTrumpAndCall(suit, card)`, `PlayCard`.
- **PlayerView:** shows only the player's own hand plus public information, and includes the legal actions. The partner stays hidden until the called ace or 4th ace is played.
