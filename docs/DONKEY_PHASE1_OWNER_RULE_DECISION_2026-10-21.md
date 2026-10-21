# Owner Rule Decision Record — 2026-10-21

**Document ID:** DM-OWNER-DECISION-2026-10-21-R008-R009  
**Date:** 2026-10-21  
**Author:** Product Owner  
**Status:** PARTIALLY APPROVED — Phase 2 **NOT AUTHORIZED**

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

## Confirmed Product Objective: WIN-001

**The first qualifying finisher takes first place and is the game winner. The last player still holding cards is the Donkey.**

A qualifying finisher is a player whose hand remains empty after full round resolution (R-009 APPROVED). A HIT-round trick winner (who collects cards per R-008) is **not** necessarily the game winner — collecting trick cards means they have not finished.

### Important Scoping Notes

- **WIN-001 is a game objective**, not a HIT-round resolution rule. Do not conflate the first-place game winner (WIN-001) with the trick winner of a single HIT round (R-008).
- **"Successfully"** is now defined by R-009: empty after full round settlement including HIT collection.
- When multiple players qualify in the same round, unique first-place priority versus tied rank is still **OPEN** (R-010/D05).
- If all active players empty simultaneously, the outcome is still **OPEN** (R-013/D06).
- Play is expected to continue to rank remaining positions, but R-010 and R-013 are **not** automatically approved by WIN-001 or R-009.

---

## Owner Decision Register

| Item | Decision | Status |
|---|---|---|
| **R-008 / D03** | HIT round won by highest card of the leading suit | **OWNER APPROVED — 2026-10-21** |
| **R-009 / D04** | Finisher identified only after round settlement, including HIT collection | **OWNER APPROVED — 2026-10-21** |
| **WIN-001** | First qualifying finisher takes first place (when unambiguous); last remaining player is Donkey | **OWNER CONFIRMED — 2026-10-21** |
| R-002 / D01 | Shuffle / deal distribution | **OPEN** |
| R-003 / D02 | A♠ holder opening-card constraint | **OPEN** |
| R-010 / D05 | Simultaneous finisher positions / ties | **OPEN** |
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

1. Mark only **R-008 and R-009** as owner-approved, and WIN-001 as owner-confirmed. Do not silently mark any other OPEN or PROPOSED decision as approved.
2. Update the canonical draft ruleset, rules evidence, test matrix, risk/decision register, Phase 1 report and owner checklist to reflect these two approvals consistently. Preserve rule version as **DRAFT** until all blocking decisions are signed.
3. Treat R-010 (same-round multi-finish) and R-013 (all-empty) as separate **OPEN** decisions.
4. No application code, tests, migrations, assets or CI workflows are authorized yet. Phase 2 remains blocked until the complete rules freeze and explicit owner authorization.

---

## Cross-References

- `docs/03-canonical-ruleset-v1-draft.md` §8 Hit Round Resolution, §9 Finisher Timing (updated)
- `docs/02-verified-rules-and-open-questions.md` HPA-3, HPA-4 (updated)
- `docs/12-risks-and-decisions.md` D03, D04 rows (updated)
- `RULE_FREEZE_APPROVAL_CHECKLIST.md` D03, D04 lines (checked)
- `docs/REMAINING_DECISIONS_WORKSHEET.md` — full sign-off worksheet for all pending items
