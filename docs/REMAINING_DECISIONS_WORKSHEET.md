# Remaining Decisions Worksheet — Phase 1

**Document ID:** DM-WORKSHEET-PENDING  
**Date:** 2026-10-21  
**Status:** ALL ITEMS RESOLVED — see summary below

---

## Resolved (for reference)

| ID | Decision | Resolution | Date |
|---|---|---|---|
| D03 / R-008 | HIT winner: leading-suit highest only | **OWNER APPROVED** — product choice, not competitor verification | 2026-10-21 |
| D04 / R-009 | Finisher timing: after round resolution | **OWNER APPROVED** — finisher identified only after full round settlement including HIT collection | 2026-10-21 |
| D05 / R-010 | Simultaneous finisher ranking: clockwise from leader | **OWNER APPROVED** — distinct consecutive positions in clockwise seat order from round leader; no tied ranks | 2026-10-21 |
| D01 / R-002 | Shuffle distribution: uniform CSPRNG | **OWNER APPROVED** — uniform Fisher-Yates CSPRNG shuffle; 13 cards each; no suit-distribution skewing or balancing heuristic; fair, simple, testable, original | 2026-10-21 |
| D02 / R-003 | A♠ opening constraint: free choice | **OWNER APPROVED** — A♠ holder starts round but may lead any card; A♠ not mandatory; "starts" means goes first | 2026-10-21 |
| D06 / R-013 | All players empty simultaneously: R-010 clockwise ranking; last = Donkey | **OWNER APPROVED** — rank all qualifying finishers by R-010 clockwise order from round leader; last-ranked finisher is Donkey; no draw, no void | 2026-10-21 |
| WIN-001 | Game objective: first qualifying finisher (per R-009) = winner | **CONFIRMED** — "successfully" now defined by R-009 (APPROVED); simultaneous finisher ranking resolved by R-010 (APPROVED); all-empty resolved by R-013 (APPROVED) | 2026-10-21 |
| D07 | Timeout / disconnect policy | **OWNER CONFIRMED** — 20s action deadline, 15s disconnect grace period, disclosed bot takeover, player can reclaim at round boundary | 2026-10-21 |
| D08 | Bot fairness: no hidden-hand access | **OWNER CONFIRMED** — bots see only their own hand and played cards (same as a human seat) | 2026-10-21 |
| D09 | MVP player count: 4 only | **OWNER CONFIRMED** — 4 players only at MVP; no 2/3-player variant until separately specified | 2026-10-21 |
| D10 | Online bot disclosure | **OWNER CONFIRMED** — bots disclosed; queue fallback only after user opt-in or timeout; all bot-controlled seats labeled | 2026-10-21 |
| D11 | 2.5D rendering | **OWNER CONFIRMED** — Flame 2.5D with authored assets; no Unity/Unreal (ADR-0001) | 2026-10-21 |
| D12 | Dart SignalR client | **OWNER CONFIRMED** — compatibility spike in Phase 4; WebSocket fallback ready (ADR-0002) | 2026-10-21 |
| D13 | Offline currency isolation | **OWNER CONFIRMED** — offline coins/progression fully separate from online server wallet | 2026-10-21 |
| D14 | Regional compliance | **OWNER CONFIRMED** — India-first launch; no wagering/cash prizes; legal review before international expansion | 2026-10-21 |
| R-017 | Watchdog threshold | **OWNER APPROVED** — void after 200 rounds or 30 minutes wall-clock time, whichever comes first; never assign arbitrary Donkey; threshold tunable post-launch | 2026-10-21 |

---

## Summary

| Category | Count | Items |
|---|---|---|
| **OWNER APPROVED** | 7 | D01/R-002, D02/R-003, D03/R-008, D04/R-009, D05/R-010, D06/R-013, R-017 |
| **OWNER CONFIRMED** | 8 | WIN-001, D07, D08, D09, D10, D11, D12, D13, D14 |
| **OPEN (blocking)** | 0 | — |
| **PROPOSED** | 0 | — |
| **Total requiring sign-off** | 0 | All items resolved |

**Rule freeze condition:** All blocking and proposed decisions are now signed. Ruleset `DM-RULES-1.0` may be issued and Phase 2 authorized at the product owner's discretion.

---

**Worksheet prepared by:** Architect (Phase 1 review)  
**Date:** 2026-10-21
