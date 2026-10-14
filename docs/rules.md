# Rikken (base game): rules spec v1

Based on the nl-BE rules in `spelregels-rikken.pdf` [1] and the v1 decisions.

## 1. Setup

- 4 players. Bots fill empty seats.
- 52 cards, no jokers. Rank runs from 2 (low) to ace (high) [1].
- 13 cards per player.
- Seeded deal. The dealer rotates clockwise.

## 2. Bidding

- The player left of the dealer bids first, then play continues clockwise. A player who has passed cannot bid again [1].
- Bids, lowest to highest: `Pass` < `Rik 8` < `Rik 9` < … < `Rik 13`.
  - A rik bid is the number of tricks the bidder and partner must win together.
  - Each bid must be higher than the previous one.
- Bidding ends once three passes have followed a bid. Passes made earlier by players who have since been skipped count towards the three, so bidding ends as soon as every player except the highest bidder has passed.
- The highest bidder therefore never gets a turn to raise their own bid.
- Out of scope: _betere rik_, _acht alleen_, _piek/piccolo_, _misère_, _troela_ and pass games. Every game is 2 vs 2.
- If everyone passes, the cards are redealt by the next dealer.

## 3. Contract set-up

### Rik

- After the others have passed, the highest bidder names the trump suit and the ace they call. The holder of that ace is the partner [1].
- The partner may not reveal themselves. They stay hidden until the called ace is played [1].
- The bidder may not call an ace they hold themselves. If the bidder holds all four aces, they call a king instead, and the holder of that king is the partner.

## 4. Play

- The player left of the dealer leads the first trick. After that, the trick winner leads.
- Players must follow suit if they can. Otherwise they may play any card, including a trump.
- The highest trump wins the trick. If no trump was played, the highest card of the led suit wins.
- 13 tricks are played. Tricks are counted per team.

## 5. Result and scoring

- The contract team wins if it takes at least the bid number of tricks.
- Scoring is always per player. Each member of the contract team receives the contract team's result. Each opposing player receives the same number of points with the opposite sign
- Contract team win: 10 + 5 per trick above 8, regardless of the bid [1].
- Contract team loss: 10 + 5 per trick short of the bid, so a loss is never below 15 [1].

| Contract | Tricks taken | Contract team | Opposing team |
| -------- | ------------ | ------------- | ------------- |
| Rik 8    | 8            | +10           | -10           |
| Rik 8    | 9            | +15           | -15           |
| Rik 10   | 10           | +20           | -20           |
| Rik 10   | 11           | +25           | -25           |
| Rik 13   | 13           | +35           | -35           |
| Rik 8    | 7            | -15           | +15           |
| Rik 10   | 8            | -20           | +20           |
| Rik 13   | 10           | -25           | +25           |

## 6. Game end

- A game is a series of deals. It ends after the first deal in which a player's running score reaches 50 or more.
- Partners change every deal, so scores are kept per player.
- If several players reach 50 in the same deal and have the same top score, all of them are winners.
- A deal where everyone passes (section 2) is redealt and does not change any score.

## 7. Engine notes

- **State:** seed, dealer, bids, contract, trump (null until the contract is set up), called ace, hidden partner, tricks, hands, running scores per player.
- **Actions:** `Bid(n)`, `Pass`, `ChooseTrumpAndCall(suit, card)`, `PlayCard`.
- **PlayerView:** shows only the player's own hand plus public information, and includes the legal actions. The partner stays hidden until the called ace is played.
