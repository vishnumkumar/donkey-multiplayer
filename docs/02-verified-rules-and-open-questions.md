# 02 — Verified Rules and Open Questions

**Document ID:** DM-DOC-02  
**Version:** 0.1 DRAFT  
**Date:** 2026-10-20  
**Status:** Awaiting owner approval — do not implement

---

## Purpose

Provide a rigorous evidence-based analysis of the Donkey card game rules. For each rule area, cite the primary source, quote relevant text, classify as CONFIRMED / PROPOSED / OPEN, and identify gaps. This document is the **evidence table** — the canonical ruleset is in [03-canonical-ruleset-v1-draft.md](03-canonical-ruleset-v1-draft.md).

## Scope

Compares publicly available rule descriptions against our proposed ruleset. Does not claim to reverse-engineer or replicate CodeHound's proprietary implementation. Ambiguities are flagged for owner decision, not silently resolved.

---

## Primary Source

| Source | URL | Type |
|---|---|---|
| CodeHound Donkey Master — How to Play | https://www.donkeymaster.in/ | Developer-published rules page |
| Apple App Store listing | https://apps.apple.com/app/id1326593908 | Store description |
| Google Play listing | https://play.google.com/store/apps/details?id=in.codehound.donkey | Store description |

**Access limitation:** This analysis is based on publicly available text from these URLs. No private protocols, source code, or proprietary content was accessed. Web lookup was not performed in this session; citations reference content known from the spec's own transcription. Where the spec's quotation of the source is the basis, this is noted.

---

## HIGH-PRIORITY AMBIGUOUS RULES

The following 12 items **must** appear on the owner approval worksheet. They are blocking decisions for Phase 2.

### HPA-1: Deck Distribution (Shuffle Policy)

| Aspect | Detail |
|---|---|
| **Source evidence** | Developer site states deals intentionally avoid equal suit distribution across players. Algorithm not published. |
| **Our proposal** | Uniform 52-card CSPRNG shuffle; deal 13 to each player. No forced suit imbalance. |
| **Classification** | **[OPEN]** |
| **Gap** | The competitor's distribution algorithm is proprietary. Implementing an unknown manipulation risks incorrect gameplay feel or legal concerns. |
| **Options** | **A (Recommended):** Uniform shuffle — fair, simple, testable, original. **B:** Light suit-balancing heuristic (e.g., ensure no player gets >7 of one suit) — closer to competitor claim but invents an unverified algorithm. **C:** Post-shuffle rejection sampling (discard deals with extreme distribution) — middle ground but adds complexity. |
| **Risk of guessing** | If competitor's distribution materially affects strategy, our game may feel different. Acceptable for an original product. |
| **Maps to** | R-002, D01 |

### HPA-2: Opening Card Constraint

| Aspect | Detail |
|---|---|
| **Source evidence** | "Holder of Ace of Spades starts the first round" — source says **starts**, not **must play A♠**. |
| **Our proposal** | A♠ holder acts first but may lead **any** held card. A♠ is not mandatory. |
| **Classification** | **[OPEN]** |
| **Gap** | "Starts" is ambiguous — could mean "starts by playing A♠" or "starts the round (plays any card)." |
| **Options** | **A (Recommended):** Any card — more strategic freedom. **B:** Must play A♠ — simpler, removes first-move choice. |
| **Risk of guessing** | Low strategic impact either way; both are common in similar card games. |
| **Maps to** | R-003, D02 |

### HPA-3: Hit Winner — Cross-Suit Comparison

| Aspect | Detail |
|---|---|
| **Source evidence** | Developer uses "largest card" without specifying whether off-suit cards participate in comparison. Example: lead 8♣, players play J♣, 10♣, A♥ — does A♥ (rank 14) beat J♣ (rank 11)? |
| **Our rule** | Highest rank **of the leading suit only** wins the trick. Off-suit (hit) cards cannot win. |
| **Classification** | **[OWNER APPROVED 2026-10-21]** — resolved by product owner decision, not by independent competitor verification. |
| **Owner decision** | "When any player makes a HIT… the highest-ranked card of the leading suit only wins the round… No off-suit card can win merely by having a greater rank." Acceptance example: lead 8♣; players then play J♣, 10♣, A♥. The A♥ is a HIT; the J♣ seat wins and collects all four cards. |
| **Distinction** | This is the product owner's choice for our original game, not independent proof that CodeHound uses it. The source ambiguity (HPA-3) remains unresolved for competitor behavior. |
| **Gap** | "Largest card" could mean largest overall (A♥ beats J♣) or largest of leading suit (J♣ wins). These produce fundamentally different gameplay. |
| **Options considered** | **A (APPROVED):** Leading-suit highest wins — standard in most trick-taking games; off-suit is a penalty (hit), not a power play. **B (Rejected):** Absolute highest rank wins regardless of suit — makes off-suit play strategically viable, radically changes hit dynamics. **C (Rejected):** Leading suit wins ties, off-suit wins only if strictly higher — hybrid; complex to explain. |
| **Risk of guessing** | N/A — resolved by owner decision. |
| **Maps to** | R-008, D03 |

### HPA-4: Finisher Timing

| Aspect | Detail |
|---|---|
| **Source evidence** | Not addressed in published rules. |
| **Our rule** | A player qualifies as a finisher **only after the entire round has resolved**, including any HIT collection and card transfers. An empty hand immediately after the last card is played does not by itself establish a finish. |
| **Classification** | **[OWNER APPROVED 2026-10-21]** — resolved by product owner decision. |
| **Owner decision** | Finisher identified only after round settlement, including HIT collection. A player who played their last card but then receives collected trick cards is not a finisher. A player still empty after settlement qualifies and exits subsequent rounds. |
| **Gap** | No explicit source guidance on when "empty hand" is evaluated. |
| **Options considered** | **A (APPROVED):** Evaluate finishers after round resolution — prevents incorrect early exit when hit returns cards. **B (Rejected):** Immediate exit upon playing last card — creates paradoxes with hit collection. |
| **Risk of guessing** | N/A — resolved by owner decision. |
| **Maps to** | R-009, D04 |

### HPA-5: Multiple Simultaneous Finishers

| Aspect | Detail |
|---|---|
| **Source evidence** | No published rule for the case where multiple players reach zero cards in the same round resolution. |
| **Our rule** | Assign **distinct, consecutive positions** in clockwise seat order starting with the completed round's leader (inclusive). Do not award tied ranks. |
| **Classification** | **[OWNER APPROVED 2026-10-21]** — resolved by product owner decision. |
| **Owner decision** | When two or more players become qualifying finishers in the same fully settled round, traverse clockwise from the round leader and assign the next available finishing position to each qualifying seat encountered. Deterministic example: leader = Seat 1; Seats 2 and 0 qualify; clockwise from leader: Seat 2 gets next position, then Seat 0. |
| **Gap** | No published source guidance; this is a product choice. |
| **Options considered** | **A (APPROVED):** Clockwise deterministic — simple, unambiguous, testable. **B (Rejected):** Tied ranks — fairer in theory but complicates podium display and leaderboard math. |
| **Risk of guessing** | N/A — resolved by owner decision. |
| **Maps to** | R-010, D05 |

### HPA-6: All Players Empty Together

| Aspect | Detail |
|---|---|
| **Source evidence** | No published rule for this edge case. |
| **Our proposal** | Technical draw — no Donkey assigned. |
| **Classification** | **[OPEN]** |
| **Gap** | Could also replay the match, assign Donkey to last player who received cards, or other resolution. |
| **Options** | **A (Recommended):** Draw/no Donkey — no unfair invented loser. **B:** Assign Donkey to last player who collected a hit — has some logic but punishes for an unavoidable game state. **C:** Replay — wastes player time for an extremely rare event. |
| **Risk of guessing** | Very low — this is an extremely rare edge case. Any consistent decision is acceptable. |
| **Maps to** | R-013, D05 |

### HPA-7: Last Active Seat Declaration

| Aspect | Detail |
|---|---|
| **Source evidence** | "Last card-holder is the Donkey" is confirmed in principle; exact trigger timing is unspecified. |
| **Our proposal** | When only one active player remains with cards after a round is fully resolved, that player is immediately declared Donkey. |
| **Classification** | **[PROPOSED]** |
| **Gap** | Does Donkey declaration happen mid-trick or only after full resolution? What if the Donkey candidate is the trick leader for an incomplete round? |
| **Options** | **A (Recommended):** Declare after full round resolution — consistent with finisher timing. **B:** Declare as soon as second-to-last player exits — could interrupt an in-progress trick. |
| **Risk of guessing** | Option A is consistent with HPA-4 finisher timing. Option B creates complex mid-trick edge cases. |
| **Maps to** | R-012 |

### HPA-8: Timeout and Disconnect Policy

| Aspect | Detail |
|---|---|
| **Source evidence** | No published timeout or disconnect rules from competitor. |
| **Our proposal** | 20-second action deadline; 15-second disconnection grace; server bot takeover with disclosure. |
| **Classification** | **[PROPOSED]** — user experience decision |
| **Gap** | All values are configurable proposals. Need product decision on: grace duration, bot takeover vs. match void, reclaim timing. |
| **Options** | **A (Recommended):** 20s/15s with bot takeover — keeps game moving for other players. **B:** Longer grace (30s/30s) — more forgiving for bad connections but frustrating for waiting players. **C:** Match pause/void on disconnect — safest but worst experience for remaining players. |
| **Risk of guessing** | Medium. Too aggressive = frustrating for users on bad networks. Too lenient = frustrating for waiting players. Should be configurable and tuned. |
| **Maps to** | D06 |

### HPA-9: Bot Fairness

| Aspect | Detail |
|---|---|
| **Source evidence** | No published bot policy from competitor. |
| **Our proposal** | Bots may not inspect opponent hidden hands. Bot decision policy restricted to information visible to the simulated seat. Bot seats are always labeled. |
| **Classification** | **[PROPOSED]** |
| **Options** | **A (Recommended):** Fair bots — same information as human player. **B:** Omniscient bots with difficulty curve — simpler to implement but unfair and deceptive. |
| **Maps to** | D10 |

### HPA-10: Player Count (MVP)

| Aspect | Detail |
|---|---|
| **Source evidence** | 4-player confirmed as standard mode. 2/3 player variants exist in competitor app per store listings but rules are not published. |
| **Our proposal** | 4 players only at MVP. 2/3-player variants require separate rule specification (dealing remainder cards, etc.). |
| **Classification** | **[PROPOSED]** — MVP scope decision |
| **Maps to** | D08 (partial) |

### HPA-11: Round Repetition Watchdog

| Aspect | Detail |
|---|---|
| **Source evidence** | No published rule for pathological infinite loops. |
| **Our proposal** | Technical watchdog may void a match after N rounds (threshold TBD). **Never** award an arbitrary Donkey. |
| **Classification** | **[OPEN]** — threshold needs testing |
| **Options** | **A (Recommended):** Void after configurable round limit (e.g., 200 rounds) — prevents infinite matches, no unfair outcome. **B:** No watchdog; trust that game always terminates — risky if edge cases exist. |
| **Maps to** | R-017 |

### HPA-12: Offline Currency Isolation

| Aspect | Detail |
|---|---|
| **Source evidence** | No published economy design from competitor. |
| **Our proposal** | Offline virtual coins/rewards are completely isolated from online server economy. No upload or sync. |
| **Classification** | **[PROPOSED]** — security requirement |
| **Options** | **A (Recommended):** Complete isolation — no tamper vector. **B:** Sync offline earnings on reconnect — complex, tamper-prone. |
| **Maps to** | D07 |

---

## COMPLETE RULES EVIDENCE TABLE

| Rule Area | Source Evidence | Source URL | Certainty | Our Proposal | Gap / Notes |
|---|---|---|---|---|---|
| **Deck composition** | Standard 52-card deck, no jokers, four suits | donkeymaster.in | **CONFIRMED** | 52 cards, 4 suits, 13 ranks (2–A) | None |
| **Rank order** | Ascending 2–10, J, Q, K, A | donkeymaster.in | **CONFIRMED** | A(14) > K(13) > … > 2(2) | None |
| **Player count** | Four players | donkeymaster.in | **CONFIRMED** | Exactly 4 seated players at match start | 2/3 player variants unspecified |
| **Deal** | Entire deck dealt evenly to four players (13 each) | donkeymaster.in | **CONFIRMED** | 13 cards each | None |
| **Suit distribution** | "Deals intentionally avoid equal suit distribution" | donkeymaster.in | **CONFIRMED** (claim exists) | Uniform shuffle — do not implement unknown algorithm | Algorithm not published; see HPA-1 |
| **Opening** | "Holder of Ace of Spades starts the first round" | donkeymaster.in | **CONFIRMED** (who starts) | A♠ holder leads first; any card legal | Whether A♠ must be played: **OPEN**; see HPA-2 |
| **Follow suit** | Players must follow lead suit if held | donkeymaster.in | **CONFIRMED** | If holding ≥1 card of lead suit, must play one | Enforcement mechanism is our design |
| **Hit definition** | Playing off-suit when void in lead suit = HIT | donkeymaster.in | **CONFIRMED** | Off-suit play when void = hit flag | None |
| **No-hit resolution** | Cards leave hands (discarded); highest card player leads next | donkeymaster.in | **CONFIRMED** (partial) | Highest leading-suit card wins; cards to discard pile | "Highest card" ambiguous across suits |
| **Hit resolution** | "Largest-card player takes all round cards" | donkeymaster.in | **CONFIRMED** (partial) | Highest leading-suit card takes all played cards | Cross-suit comparison: **OPEN**; see HPA-3 |
| **Hit leader** | Winner of hit round leads next | donkeymaster.in | **CONFIRMED** | Hit winner leads next round | None |
| **Finishing** | "Players try to exhaust cards; finishers are ranked" | donkeymaster.in | **CONFIRMED** (objective) | Finishers ranked; podium positions assigned | Timing: **PROPOSED** (after resolution) |
| **Donkey** | "Last card-holder is the Donkey" | donkeymaster.in | **CONFIRMED** (objective) | Last remaining player with cards = Donkey | Exact trigger: **PROPOSED** |
| **Turn order** | Clockwise | donkeymaster.in (implied) | **PROPOSED** | Clockwise from leader; skip finished players | Not explicitly stated but standard |
| **Simultaneous finish** | Not addressed | — | **OPEN** | Clockwise deterministic ranking | See HPA-5 |
| **All empty** | Not addressed | — | **OPEN** | Draw / no Donkey | See HPA-6 |
| **Finisher-exits-as-leader** | Not addressed | — | **PROPOSED** | Next active clockwise seat leads | — |
| **Timeout** | Not addressed | — | **PROPOSED** | 20s deadline, 15s grace, bot takeover | See HPA-8 |
| **Disconnect** | Not addressed | — | **PROPOSED** | Reconnect with snapshot; bot during grace | See HPA-8 |
| **Bot behavior** | Not addressed | — | **PROPOSED** | No hidden-hand access; labeled seats | See HPA-9 |
| **Watchdog** | Not addressed | — | **OPEN** | Void after N rounds; no arbitrary Donkey | See HPA-11 |

---

## Source Quotation Summary

The following is a paraphrase summary of the rules as described on the developer site (https://www.donkeymaster.in/), based on the spec's own transcription:

1. One normal 52-card deck, no jokers.
2. Four suits; ascending rank 2–10, J, Q, K, A.
3. Entire deck dealt evenly to four players (13 each).
4. Holder of Ace of Spades starts the first round.
5. Players follow lead suit if they possess it; otherwise play another suit — called a HIT.
6. When there is no hit, the largest card of the round leads next and round cards leave hands.
7. When there is a hit, the largest-card player takes all round cards back into their hand and leads next.
8. Players try to exhaust their cards; finishers are ranked.
9. Last card-holder is the Donkey.
10. Deals intentionally avoid equal suit distribution across players (algorithm not published).

**What is NOT specified by the source:**

- Whether "largest card" means largest of leading suit or largest across all suits
- Whether A♠ must be the opening card or just its holder starts
- What happens when multiple players finish simultaneously
- What happens when all players empty their hands at once
- Exact turn-order definition (clockwise assumed but not stated)
- Finisher timing relative to round resolution
- Timeout, disconnect, and reconnection behavior
- Bot behavior and fairness constraints
- Watchdog/infinite-loop policy
- Economy or monetization rules

---

## Acceptance Criteria for This Document

- [ ] Every rule area has a source citation or explicit "not addressed"
- [ ] All 12 high-priority ambiguities are prominently documented with options
- [ ] No ambiguity is silently resolved as if it were confirmed
- [ ] CONFIRMED items have verifiable source references
- [ ] OPEN items have clear options with recommendations
- [ ] Owner can use this document to make informed decisions on all blocking items

---

## Cross-References

| Document | Relationship |
|---|---|
| [03-canonical-ruleset-v1-draft.md](03-canonical-ruleset-v1-draft.md) | Canonical ruleset built from these evidence findings |
| [10-qa-test-matrix-and-acceptance.md](10-qa-test-matrix-and-acceptance.md) | Test fixtures derived from confirmed and proposed rules |
| [12-risks-and-decisions.md](12-risks-and-decisions.md) | Decision register for all OPEN items |
| `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT.md` | Source baseline containing original evidence transcription |
| `RULE_FREEZE_APPROVAL_CHECKLIST.md` | Approval checklist for all blocking decisions |
