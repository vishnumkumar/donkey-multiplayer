# Owner Rule Decision Record — 2026-10-21

**Document ID:** DM-OWNER-DECISION-2026-10-21-R008  
**Date:** 2026-10-21  
**Author:** Product Owner  
**Status:** RECORDED — applies to Phase 1 documentation only; no code implemented

---

## Decision: R-008 / D03 — HIT Winner Determination

**APPROVED** for our original game.

When any player makes a HIT (plays off-suit because they have no cards of the leading suit), the highest-ranked card **of the leading suit only** wins the round. All played cards (including the off-suit HIT cards) are collected into the winner's hand. No off-suit card can win merely by having a greater rank.

### Acceptance Example

Lead 8♣; players then play J♣, 10♣, A♥. The A♥ is a HIT; the J♣ seat wins and collects all four cards.

### Distinction from Competitor Behavior

This is the product owner's choice for our original game, **not** independent proof that CodeHound uses this rule. The competitor's published description ("largest card") does not specify whether cross-suit comparison applies. Their actual implementation behavior is **NOT VERIFIED** — we do not claim to know or reproduce it.

### Mapping

| ID Scheme | ID | Status |
|---|---|---|
| Decision Register | D03 | OWNER APPROVED |
| Canonical Ruleset | R-008 | OWNER APPROVED |
| Rules Evidence | HPA-3 | OWNER APPROVED (source ambiguity remains for competitor) |

---

## Confirmed Product Objective: WIN-001

**The first player who successfully empties their entire hand is the first-place game winner. The last player still holding cards is the Donkey.**

### Important Scoping Notes

- **"Successfully"** depends on finisher timing (R-009): a player who plays their last card off-suit in a HIT may receive cards back into their hand. R-009 is **NOT APPROVED** — it remains PROPOSED with the recommended default of "after round resolution."
- **WIN-001 is a game objective**, not a HIT-round resolution rule. Do not conflate the first-place game winner (WIN-001) with the trick winner of a single HIT round (R-008).
- Whether play continues to rank other finishers, how to classify simultaneous finishers (R-010/D05), and the all-empty draw scenario (R-013/D06) are **NOT** automatically approved by the first-to-empty statement.
- R-010 (simultaneous finisher ranking) and R-013 (all-empty draw) remain **OPEN** pending separate owner decisions.

---

## What Is NOT Approved

The following items require separate explicit owner decisions:

| ID | Description | Current Status |
|---|---|---|
| R-002 / D01 | Shuffle distribution policy | OPEN |
| R-003 / D02 | A♠ opening constraint | OPEN |
| R-009 / D04 | Finisher timing (after resolution vs. immediate) | PROPOSED |
| R-010 / D05 | Simultaneous finisher ranking | OPEN |
| R-013 / D06 | All players empty simultaneously | OPEN |
| R-017 | Watchdog threshold | OPEN |
| D07 | Timeout / disconnect / bot takeover policy | PROPOSED |
| D08 | Bot fairness — no hidden hand access | PROPOSED |

These cannot be inferred from the R-008 or WIN-001 decisions.

---

## Cross-References

- `docs/03-canonical-ruleset-v1-draft.md` §8 — Hit Round Resolution (updated)
- `docs/02-verified-rules-and-open-questions.md` HPA-3 (updated)
- `docs/12-risks-and-decisions.md` D03 row (updated)
- `RULE_FREEZE_APPROVAL_CHECKLIST.md` D03 line (checked)
- `docs/REMAINING_DECISIONS_WORKSHEET.md` — full sign-off worksheet for all pending items
