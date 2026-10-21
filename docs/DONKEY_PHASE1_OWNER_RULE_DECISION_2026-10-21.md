# Owner Rule Decision Record — 2026-10-21

**Document ID:** DM-OWNER-DECISION-2026-10-21-R008-R009-R010  
**Date:** 2026-10-21  
**Author:** Product Owner  
**Status:** PARTIALLY APPROVED (R-008, R-009, R-010) — Phase 2 **NOT AUTHORIZED**

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
4. **Multiple same-round finishers:** If multiple players qualify in the same round, **R-010 remains OPEN** — do not assign unique first place based on arbitrary seat order until the owner chooses ties versus ranking.
5. **All players empty:** If all active players empty in the same resolution, **R-013 remains OPEN** — do not infer the draw rule from R-009 approval.

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
- **R-013 remains OPEN:** If ALL active players empty simultaneously, the all-empty outcome (draw vs. other) is a separate decision. R-010 does not resolve it.
- **WIN-001:** The first qualifying finisher (position 1) is the game winner. R-010 determines who gets position 1 when multiple players qualify in the same round.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D05 | OWNER APPROVED |
| Canonical Ruleset | R-010 | OWNER APPROVED |
| Rules Evidence | HPA-5 | OWNER APPROVED |

---

## Confirmed Product Objective: WIN-001

**The first qualifying finisher takes first place and is the game winner. The last player still holding cards is the Donkey.**

A qualifying finisher is a player whose hand remains empty after full round resolution (R-009 APPROVED). A HIT-round trick winner (who collects cards per R-008) is **not** necessarily the game winner — collecting trick cards means they have not finished.

### Important Scoping Notes

- **WIN-001 is a game objective**, not a HIT-round resolution rule. Do not conflate the first-place game winner (WIN-001) with the trick winner of a single HIT round (R-008).
- **"Successfully"** is now defined by R-009: empty after full round settlement including HIT collection.
- When multiple players qualify in the same round, they are ranked by clockwise seat order from the round leader per **R-010 (OWNER APPROVED 2026-10-21)**.
- If all active players empty simultaneously, the outcome is still **OPEN** (R-013/D06).
- Play is expected to continue to rank remaining positions; R-013 is **not** automatically approved by WIN-001, R-009, or R-010.

---

## Owner Decision Register

| Item | Decision | Status |
|---|---|---|
| **R-008 / D03** | HIT round won by highest card of the leading suit | **OWNER APPROVED — 2026-10-21** |
| **R-009 / D04** | Finisher identified only after round settlement, including HIT collection | **OWNER APPROVED — 2026-10-21** |
| **R-010 / D05** | Simultaneous finishers ranked clockwise from leader; distinct consecutive positions; no ties | **OWNER APPROVED — 2026-10-21** |
| **WIN-001** | First qualifying finisher takes first place; last remaining player is Donkey | **OWNER CONFIRMED — 2026-10-21** |
| R-002 / D01 | Shuffle / deal distribution | **OPEN** |
| R-003 / D02 | A♠ holder opening-card constraint | **OPEN** |
| R-013 / D06 | All players empty at once | **OPEN** |
| R-017 | Match watchdog threshold / void policy | **OPEN** |
| D07 | Timeout/disconnect/bot takeover timing | **PENDING CONFIRMATION** |
| D08 | Bot fairness / hidden-hand policy | **PENDING CONFIRMATION** |
| D09 | Four-seat MVP scope | **PENDING CONFIRMATION** |
| D10 | Disclosure of online bot participation | **PENDING CONFIRMATION** |
| D11 | 2.5D rendering | **PENDING CONFIRMATION** |
| D12 | Flutter SignalR compatibility spike | **PENDING CONFIRMATION / TECHNICAL SPIKE** |
| D13 | Offline currency isolated from online | **PENDING CONFIRMATION** |
| D14 | India-first launch / legal review | **PENDING CONFIRMATION** |

---

## Documentation and Implementation Guardrails

1. Mark only **R-008, R-009, and R-010** as owner-approved, and WIN-001 as owner-confirmed. Do not silently mark any other OPEN or PROPOSED decision as approved.
2. Update the canonical draft ruleset, rules evidence, test matrix, risk/decision register, Phase 1 report and owner checklist to reflect these three approvals consistently. Preserve rule version as **DRAFT** until all blocking decisions are signed.
3. Treat R-013 (all-empty) as a separate **OPEN** decision. R-010 does not resolve the all-empty outcome.
4. No application code, tests, migrations, assets or CI workflows are authorized yet. Phase 2 remains blocked until the complete rules freeze and explicit owner authorization.

---

## Cross-References

- `docs/03-canonical-ruleset-v1-draft.md` §8 Hit Round Resolution, §9 Finisher Timing, §10 Simultaneous Finisher Ranking (updated)
- `docs/02-verified-rules-and-open-questions.md` HPA-3, HPA-4, HPA-5 (updated)
- `docs/12-risks-and-decisions.md` D03, D04, D05 rows (updated)
- `docs/04-game-state-machine.md` §1.3 WIN-001, §6.1 finisher determination (updated)
- `docs/10-qa-test-matrix-and-acceptance.md` FD-004, FD-005 (updated)
- `docs/PHASE_1_REVIEW_REPORT.md` §3, §5, §10.1, §11 (updated)
- `RULE_FREEZE_APPROVAL_CHECKLIST.md` D03, D04, D05 lines (checked)
- `docs/REMAINING_DECISIONS_WORKSHEET.md` — D05 moved to resolved; sign-off worksheet for remaining pending items
