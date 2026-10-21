# Owner Rule Decision Record — 2026-10-21

**Document ID:** DM-OWNER-DECISION-2026-10-21-R002-R003-R008-R009-R010-R013-D07-D14  
**Date:** 2026-10-21  
**Author:** Product Owner  
**Status:** ALL DECISIONS RESOLVED — APPROVED (R-002, R-003, R-008, R-009, R-010, R-013, R-017) + CONFIRMED (D07–D14) — Rule freeze ready

---

## Decision: R-002 / D01 — Shuffle Distribution Policy

**OWNER APPROVED 2026-10-21.**

The deck is shuffled using a **uniform random permutation (Fisher-Yates algorithm)** with a cryptographically secure random number generator (CSPRNG) for production play. Each of the 4 players receives exactly 13 cards. **No suit-distribution skewing or balancing heuristic is applied.** The shuffle is fair, simple, testable, and original.

### Golden Scenario

A standard 52-card deck is shuffled uniformly at random. Each of the 4! × C(52,13) × C(39,13) × C(26,13) possible deals is equally likely. A player may receive anywhere from 0 to 13 cards of a single suit — this is a natural outcome of uniform randomness, not a defect.

### Distinction from Competitor Behavior

The competitor's published description states that deals "intentionally avoid equal suit distribution." The competitor's distribution algorithm is **proprietary and unpublished**. This approval is the product owner's choice for our original game — we do not replicate or approximate the competitor's unknown algorithm.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D01 | OWNER APPROVED |
| Canonical Ruleset | R-002 | OWNER APPROVED |
| Rules Evidence | HPA-1 | OWNER APPROVED (source ambiguity remains for competitor) |

---

## Decision: R-003 / D02 — A♠ Opening Constraint

**OWNER APPROVED 2026-10-21.**

The player holding the Ace of Spades (A♠) leads the first round. The A♠ holder may lead with **any** card from their hand. Playing A♠ is legal but not required.

### Golden Scenario

Player at Seat 2 holds A♠ along with other cards. Seat 2 is identified as the first-round leader. Seat 2 may choose to lead with 7♦ (or any other card in hand) — they are not forced to play A♠. The suit of the card they choose becomes the leading suit for the first round.

### Distinction from Competitor Behavior

The competitor's published description says "holder of Ace of Spades starts the first round." The word "starts" is ambiguous — it could mean "starts by playing A♠" or "starts the round (plays any card)." The competitor's actual implementation behavior is **NOT VERIFIED**. This is the product owner's choice for our original game.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D02 | OWNER APPROVED |
| Canonical Ruleset | R-003 | OWNER APPROVED |
| Rules Evidence | HPA-2 | OWNER APPROVED (source ambiguity remains for competitor) |

---

## Decision: R-008 / D03 — HIT Winner Determination

**OWNER APPROVED 2026-10-21.**

If at least one legal off-suit HIT occurs in a round, the player who played the **highest-ranked card of the leading suit** wins that round and **collects all cards played during the round**. Off-suit HIT cards cannot win even when their numerical rank is greater. This is the approved rule for our *original* Donkey game; it is **not verification of CodeHound's unpublished cross-suit comparison behavior**.

### Golden Scenario

Lead 8♣, then J♣, 10♣, A♥ (legal HIT because holder has no clubs): J♣ collects all four cards, not A♥.

### Distinction from Competitor Behavior

This is the product owner's choice for our original game, **not** independent proof that CodeHound uses this rule. The competitor's published description ("largest card") does not specify whether cross-suit comparison applies. Their actual implementation behavior is **NOT VERIFIED** — we do not claim to know or reproduce it.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D03 | OWNER APPROVED |
| Canonical Ruleset | R-008 | OWNER APPROVED |
| Rules Evidence | HPA-3 | OWNER APPROVED (source ambiguity remains for competitor) |

---

## Decision: R-009 / D04 — Finisher Timing

**OWNER APPROVED 2026-10-21.**

A player qualifies as a finisher **only after the entire round has resolved**, including any HIT collection and card transfers. An empty hand immediately after the last card is played does **not** by itself establish a finish. If the player's hand remains empty after settlement, they qualify as a finisher and leave subsequent turns. If they collect the HIT trick and acquire cards again, they do not finish yet.

### Golden Scenarios

1. **HIT winner plays last card but collects:** A player holds only J♣, plays it, and a later player with no clubs plays A♥. Under R-008 the J♣ player wins the HIT and receives all trick cards. **They have not finished** under R-009.
2. **No-HIT round, last card discarded:** A player plays their last card; the round has no HIT and all played cards are discarded. The player's hand remains empty. **They qualify as a finisher** after round resolution.
3. **Off-suit HIT, another player collects:** A player plays their last card as an off-suit HIT, while another player wins collection with the highest leading-suit card. After resolution, the hitting player's hand remains empty. **They qualify as a finisher**.
4. **Multiple same-round finishers:** If multiple players qualify in the same round, **R-010 (OWNER APPROVED 2026-10-21)** applies — distinct positions in clockwise order from the round leader.
5. **All players empty:** If all active players empty in the same resolution, **R-013 (OWNER APPROVED 2026-10-21)** applies — rank all finishers by R-010 clockwise order from the round leader; the last-ranked finisher is Donkey.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D04 | OWNER APPROVED |
| Canonical Ruleset | R-009 | OWNER APPROVED |
| Rules Evidence | HPA-4 | OWNER APPROVED |

---

## Decision: R-010 / D05 — Simultaneous Finisher Ranking

**OWNER APPROVED 2026-10-21.**

When two or more players become qualifying finishers in the same fully settled round (per R-009), assign **distinct, consecutive finishing positions** in **clockwise seat order** starting from the completed round's leader (inclusive). No tied ranks are awarded.

### Deterministic Algorithm

After round resolution and all card movement (per R-009):
1. Identify all active players whose hand is now empty — these are the round's qualifying finishers.
2. Starting from the round leader's seat, traverse clockwise through all seats.
3. For each seat encountered that is a qualifying finisher, assign the next available finishing position.
4. Players who already finished in prior rounds are skipped (they already hold a position).
5. Active players who still hold cards are skipped.

### Golden Scenarios

1. **Two finishers, leader S1:** Seats S2 and S3 both empty after resolution. Clockwise from S1: S2 gets next position, S3 gets the one after. S0 and S1 still have cards — play continues.
2. **Two finishers, leader S3:** Seats S0 and S2 both empty after resolution. Clockwise from S3: S0 gets next position, S2 gets the one after. Different leader changes the ordering.
3. **HIT prevents finish:** S1 plays last card (leading suit), collects the HIT trick → S1 now has cards → NOT a finisher. S3 played last card off-suit → hand empty after resolution → S3 finishes alone. R-010 does not apply when only one player qualifies.

### Rules Integration

- **R-009 prerequisite:** Finisher status is evaluated only after full round settlement. R-010 applies to the set of qualifying finishers identified by R-009.
- **R-013 (APPROVED):** If ALL active players empty simultaneously, rank all qualifying finishers by R-010 clockwise order from the round leader; the last-ranked finisher is Donkey.
- **WIN-001:** The first qualifying finisher (position 1) is the game winner. R-010 determines who gets position 1 when multiple players qualify in the same round.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D05 | OWNER APPROVED |
| Canonical Ruleset | R-010 | OWNER APPROVED |
| Rules Evidence | HPA-5 | OWNER APPROVED |

---

## Decision: R-013 / D06 — All Players Empty Simultaneously

**OWNER APPROVED 2026-10-21.**

When all remaining active players empty their hands after the same completed round, first resolve all card movements under R-008/R-009, then rank all qualifying finishers using the approved R-010 clockwise order starting from the completed round's leader. The **last-ranked finisher is Donkey**. All other finishing positions and the WIN-001 first-place winner are preserved. This outcome is not classified as a draw or void.

### Golden Scenarios

1. **Two remaining players empty (leader S1):** S0 finished 1st, S2 finished 2nd. S1 and S3 each play their last card; no hit; cards discarded. After resolution, both hands empty. R-010 clockwise from leader S1: S1 → S3. S1 → 3rd; S3 → 4th (Donkey). 52-card conservation: all hands(0) + trick(0) + discard(52) = 52. ✓
2. **Two remaining players empty (leader S3):** Same setup but leader S3. R-010 clockwise from S3: S3 → S1. S3 → 3rd; S1 → 4th (Donkey). Different leader reverses Donkey assignment.
3. **All four players empty (leader S0):** Each player has one card; all follow suit; no hit. R-010 clockwise from S0: S0 → S1 → S2 → S3. S0 → 1st (winner); S3 → 4th (Donkey).
4. **All four players empty (leader S2):** Same setup but leader S2. R-010 clockwise from S2: S2 → S3 → S0 → S1. S2 → 1st (winner); S1 → 4th (Donkey). Different leader changes all positions.

### Distinction from Alternatives

**Alternative A (Rejected):** Technical draw / no Donkey. Rejected — the game should always produce a Donkey when it reaches completion.

**Alternative B (Rejected):** Last player to have received cards from a hit round is Donkey. Rejected — punishes for an unavoidable game state and requires tracking additional history.

**Alternative C (Rejected):** Replay the match. Rejected — wastes player time for an extremely rare event.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D06 | OWNER APPROVED |
| Canonical Ruleset | R-013 | OWNER APPROVED |
| Rules Evidence | HPA-6 | OWNER APPROVED |

---

## Decision: R-017 — Watchdog Threshold

**OWNER APPROVED 2026-10-21.**

If a match exceeds **200 rounds** or **30 minutes** wall-clock duration from match start, whichever comes first, the match is **voided**. No Donkey is assigned. No competitive outcome is recorded. No rewards or penalties are granted. A diagnostic log is preserved. The threshold is tunable post-launch.

### Golden Scenario

A match has proceeded for 199 rounds with 2 active players still holding cards. Round 200 begins and completes normally. After round resolution, the watchdog detects roundCount = 200 ≥ WATCHDOG_ROUND_LIMIT. The match transitions to `VOIDED`. Neither player is declared Donkey. No rewards or penalties apply.

### Distinction from Alternatives

**Alternative (Rejected):** No watchdog; trust that the game always terminates naturally. Rejected — a bug or edge case could create an infinite loop, locking players and consuming server resources.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Canonical Ruleset | R-017 | OWNER APPROVED |

---

## Confirmed Default: D07 — Timeout / Disconnect Policy

**OWNER CONFIRMED 2026-10-21.**

20-second action deadline. 15-second disconnection grace period. Disclosed bot takeover after timeout. Player may reclaim their seat at the next round boundary. These values are tunable post-launch.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D07 | OWNER CONFIRMED |
| Rules Evidence | HPA-7 | OWNER CONFIRMED |

---

## Confirmed Default: D08 — Bot Fairness

**OWNER CONFIRMED 2026-10-21.**

Bots (offline or online) may **not** inspect hidden opponent hands. Bot decision logic is restricted to information visible to the simulated seat: the bot's own hand and all played cards. This applies to all difficulty levels.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D08 | OWNER CONFIRMED |
| Rules Evidence | HPA-8 | OWNER CONFIRMED |

---

## Confirmed Default: D09 — MVP Player Count

**OWNER CONFIRMED 2026-10-21.**

4 players only at MVP. No 2-player or 3-player variant until separately specified. Private rooms require 4 human or declared-bot seats.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D09 | OWNER CONFIRMED |
| Rules Evidence | HPA-9 | OWNER CONFIRMED |

---

## Confirmed Default: D10 — Online Bot Disclosure

**OWNER CONFIRMED 2026-10-21.**

All bot-controlled seats are labeled. Bots are used as queue fallback only after user opt-in or timeout. No silent bot fill.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D10 | OWNER CONFIRMED |
| Rules Evidence | HPA-10 | OWNER CONFIRMED |

---

## Confirmed Default: D11 — 2.5D Rendering

**OWNER CONFIRMED 2026-10-21.**

Flame 2.5D with authored assets. No Unity or Unreal engine. See ADR-0001 for technical basis.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D11 | OWNER CONFIRMED |
| Rules Evidence | HPA-11 | OWNER CONFIRMED |

---

## Confirmed Default: D12 — Dart SignalR Client

**OWNER CONFIRMED 2026-10-21.**

Compatibility spike against .NET 10 SignalR in Phase 4 before committing. WebSocket fallback ready if the spike fails. See ADR-0002 for technical basis.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D12 | OWNER CONFIRMED |
| Rules Evidence | HPA-12 | OWNER CONFIRMED |

---

## Confirmed Default: D13 — Offline Currency Isolation

**OWNER CONFIRMED 2026-10-21.**

Offline coins and progression are fully separate from the online server wallet. No upload or sync of offline earnings. This prevents exploitation via device clock or save manipulation.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D13 | OWNER CONFIRMED |

---

## Confirmed Default: D14 — Regional Compliance

**OWNER CONFIRMED 2026-10-21.**

India-first launch with a non-gambling economy. No wagering, no cash prizes, no transferable currency. Legal review required before international expansion.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D14 | OWNER CONFIRMED |

---

## Confirmed Product Objective: WIN-001

**The first qualifying finisher takes first place and is the game winner. The last player still holding cards is the Donkey.**

A qualifying finisher is a player whose hand remains empty after full round resolution (R-009 APPROVED). A HIT-round trick winner (who collects cards per R-008) is **not** necessarily the game winner — collecting trick cards means they have not finished.

### Important Scoping Notes

- **WIN-001 is a game objective**, not a HIT-round resolution rule. Do not conflate the first-place game winner (WIN-001) with the trick winner of a single HIT round (R-008).
- **"Successfully"** is now defined by R-009: empty after full round settlement including HIT collection.
- When multiple players qualify in the same round, they are ranked by clockwise seat order from the round leader per **R-010 (OWNER APPROVED 2026-10-21)**.
- If all active players empty simultaneously, rank all finishers by R-010 clockwise order from the round leader; the last-ranked finisher is Donkey per **R-013 (OWNER APPROVED 2026-10-21)**.
- Play is expected to continue to rank remaining positions until the Donkey is determined.

---

## Owner Decision Register

| Item | Decision | Status |
|---|---|---|
| **R-002 / D01** | Uniform CSPRNG Fisher-Yates shuffle; no suit-distribution skewing | **OWNER APPROVED — 2026-10-21** |
| **R-008 / D03** | HIT round won by highest card of the leading suit | **OWNER APPROVED — 2026-10-21** |
| **R-009 / D04** | Finisher identified only after round settlement, including HIT collection | **OWNER APPROVED — 2026-10-21** |
| **R-010 / D05** | Simultaneous finishers ranked clockwise from leader; distinct consecutive positions; no ties | **OWNER APPROVED — 2026-10-21** |
| **WIN-001** | First qualifying finisher takes first place; last remaining player is Donkey | **OWNER CONFIRMED — 2026-10-21** |
| **R-003 / D02** | A♠ holder may lead any card; A♠ not mandatory | **OWNER APPROVED — 2026-10-21** |
| **R-013 / D06** | All players empty: rank by R-010 clockwise order; last-ranked finisher is Donkey | **OWNER APPROVED — 2026-10-21** |
| **D07** | Timeout: 20s deadline, 15s grace, disclosed bot takeover, reclaim at round boundary | **OWNER CONFIRMED — 2026-10-21** |
| **D08** | Bot fairness: no hidden-hand access; bots see only own hand + played cards | **OWNER CONFIRMED — 2026-10-21** |
| **D09** | MVP: 4 players only; no 2/3-player variant | **OWNER CONFIRMED — 2026-10-21** |
| **D10** | Online bots: disclosed, queue fallback only, all bot seats labeled | **OWNER CONFIRMED — 2026-10-21** |
| **D11** | 2.5D rendering: Flame with authored assets (ADR-0001) | **OWNER CONFIRMED — 2026-10-21** |
| **D12** | Dart SignalR: compatibility spike Phase 4; WebSocket fallback (ADR-0002) | **OWNER CONFIRMED — 2026-10-21** |
| **D13** | Offline currency fully isolated from online server wallet | **OWNER CONFIRMED — 2026-10-21** |
| **D14** | India-first launch; no wagering; legal review before international | **OWNER CONFIRMED — 2026-10-21** |
| **R-017** | Watchdog: void after 200 rounds or 30 min; no arbitrary Donkey | **OWNER APPROVED — 2026-10-21** |

---

## Documentation and Implementation Guardrails

1. Mark **R-002, R-003, R-008, R-009, R-010, R-013, and R-017** as owner-approved; **D07, D08, D09, D10, D11, D12, D13, D14** as owner-confirmed (recommended defaults); and **WIN-001** as owner-confirmed objective.
2. Update the canonical draft ruleset, rules evidence, test matrix, risk/decision register, Phase 1 report and owner checklist to reflect all 15 approvals/confirmations consistently. Preserve rule version as **DRAFT** until the owner issues `DM-RULES-1.0`.
3. All blocking and proposed decisions are now resolved. Rule freeze (`DM-RULES-1.0`) and Phase 2 authorization may proceed at the product owner's discretion.
4. No application code, tests, migrations, assets or CI workflows are authorized yet. Phase 2 requires explicit owner authorization after rule freeze.

---

## Cross-References

- `docs/03-canonical-ruleset-v1-draft.md` §2 Shuffle and Deal, §4 Opening, §8 Hit Round Resolution, §9 Finisher Timing, §10 Simultaneous Finisher Ranking, §12 All Players Empty (updated)
- `docs/02-verified-rules-and-open-questions.md` HPA-1, HPA-2, HPA-3, HPA-4, HPA-5, HPA-6 (updated)
- `docs/12-risks-and-decisions.md` D01–D06 rows (approved), D07–D14 rows (confirmed) (updated)
- `docs/04-game-state-machine.md` §1.3 WIN-001, §6.1 finisher determination (updated)
- `docs/10-qa-test-matrix-and-acceptance.md` FD-004, FD-005, FD-006, FD-008 (updated)
- `docs/PHASE_1_REVIEW_REPORT.md` §3, §5, §10.1, §11 (updated)
- `RULE_FREEZE_APPROVAL_CHECKLIST.md` D01–D06 lines (approved), D07–D14 lines (confirmed) (checked)
- `docs/REMAINING_DECISIONS_WORKSHEET.md` — D07–D14 moved to resolved; R-017 is sole remaining item
