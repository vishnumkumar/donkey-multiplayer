# Remaining Decisions Worksheet — Phase 1

**Document ID:** DM-WORKSHEET-PENDING  
**Date:** 2026-10-21  
**Status:** PENDING OWNER SIGN-OFF — all items below must be resolved before DM-RULES-1.0 and Phase 2

---

## Resolved (for reference)

| ID | Decision | Resolution | Date |
|---|---|---|---|
| D03 / R-008 | HIT winner: leading-suit highest only | **OWNER APPROVED** — product choice, not competitor verification | 2026-10-21 |
| D04 / R-009 | Finisher timing: after round resolution | **OWNER APPROVED** — finisher identified only after full round settlement including HIT collection | 2026-10-21 |
| D05 / R-010 | Simultaneous finisher ranking: clockwise from leader | **OWNER APPROVED** — distinct consecutive positions in clockwise seat order from round leader; no tied ranks | 2026-10-21 |
| WIN-001 | Game objective: first qualifying finisher (per R-009) = winner | **CONFIRMED** — "successfully" now defined by R-009 (APPROVED); simultaneous finisher ranking resolved by R-010 (APPROVED); R-013 remains OPEN | 2026-10-21 |

---

## OPEN — Blocking Gameplay Decisions

These require explicit owner approval. Each has a recommended default and alternatives.

### 1. D01 / R-002 — Shuffle Distribution Policy

| | |
|---|---|
| **Question** | Uniform CSPRNG 52-card shuffle, or intentional suit-distribution skewing? |
| **Recommended** | Uniform shuffle; no forced imbalance. Fair, simple, testable, original. |
| **Alternative A** | Replicate competitor's undisclosed suit-skewing algorithm (proprietary; unknown). |
| **Alternative B** | Disclosed balancing heuristic (e.g., guarantee each player gets ≥2 of each suit). |
| **Risk if wrong** | MEDIUM — affects game feel, not correctness. Can be patched post-launch. |
| **Maps to** | R-002, HPA-1 |

**Owner decision:** ☐ APPROVE recommended ☐ SPECIFY alternative: ____________  
**Signed:** __________________ **Date:** __________

---

### 2. D02 / R-003 — A♠ Opening Constraint

| | |
|---|---|
| **Question** | A♠ holder leads any card, or must lead A♠ specifically? |
| **Recommended** | Free choice — A♠ holder starts the round but may lead any held card. |
| **Alternative** | Must play A♠ as the first card. |
| **Risk if wrong** | HIGH — affects first-round strategy for every game. |
| **Maps to** | R-003, HPA-2 |

**Owner decision:** ☐ APPROVE recommended ☐ SPECIFY alternative: ____________  
**Signed:** __________________ **Date:** __________

---

### 3. D06 / R-013 — All Players Empty Simultaneously

| | |
|---|---|
| **Question** | What happens if every player runs out of cards in the same round? |
| **Recommended** | Technical draw / no Donkey. No invented loser. |
| **Alternative A** | Replay the match. |
| **Alternative B** | Last player to have played a card is Donkey. |
| **Risk if wrong** | LOW (rare edge case) — but must be defined for determinism. |
| **Maps to** | R-013, HPA-6 |

**Owner decision:** ☐ APPROVE recommended ☐ SPECIFY alternative: ____________  
**Signed:** __________________ **Date:** __________

---

### 4. R-017 — Watchdog Threshold

| | |
|---|---|
| **Question** | At what threshold should the server void a pathological match? |
| **Recommended** | Void after 200 rounds OR 30 minutes wall-clock time, whichever comes first. Never assign arbitrary Donkey. |
| **Alternative** | Different threshold values, or no watchdog at all. |
| **Risk if wrong** | LOW — protects against infinite loops. Threshold is tunable post-launch. |
| **Maps to** | R-017 |

**Owner decision:** ☐ APPROVE recommended ☐ SPECIFY threshold: ____________  
**Signed:** __________________ **Date:** __________

---

## PROPOSED — Safe Defaults (Confirm or Override)

These have recommended defaults that are safe to implement. Owner should confirm or override.

### 5. D07 — Timeout / Disconnect Policy

| | |
|---|---|
| **Question** | What happens when a player disconnects or times out? |
| **Recommended** | 20s action deadline, 15s disconnect grace period, disclosed bot takeover, player can reclaim at round boundary. |
| **Alternatives** | Shorter/longer timers; immediate forfeit; match pause for all players. |
| **Risk if wrong** | MEDIUM — UX sensitive; can be tuned post-launch. |

**Owner decision:** ☐ CONFIRM recommended ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 6. D08 — Bot Fairness

| | |
|---|---|
| **Question** | Can bots (offline or online) inspect hidden opponent hands? |
| **Recommended** | No — bots see only their own hand and played cards (same as a human seat). |
| **Alternative** | Allow offline bots to see all hands for harder difficulty levels. |
| **Risk if wrong** | MEDIUM — player trust issue if bots cheat; perceived unfairness. |

**Owner decision:** ☐ CONFIRM recommended ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 7. D09 — MVP Player Count

| | |
|---|---|
| **Recommended** | 4 players only at MVP; no 2/3-player variant until separately specified. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 8. D10 — Online Bot Disclosure

| | |
|---|---|
| **Recommended** | Bots disclosed; queue fallback only after user opt-in or timeout; all bot-controlled seats labeled. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 9. D11 — 2.5D Rendering

| | |
|---|---|
| **Recommended** | Flame 2.5D with authored assets; no Unity/Unreal. See ADR-0001. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 10. D12 — Dart SignalR Client

| | |
|---|---|
| **Recommended** | Compatibility spike in Phase 4; WebSocket fallback ready. See ADR-0002. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 11. D13 — Offline Currency Isolation

| | |
|---|---|
| **Recommended** | Offline coins/progression fully separate from online server wallet. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

### 12. D14 — Regional Compliance

| | |
|---|---|
| **Recommended** | India-first launch; no wagering/cash prizes; legal review before international expansion. |

**Owner decision:** ☐ CONFIRM ☐ OVERRIDE: ____________  
**Signed:** __________________ **Date:** __________

---

## Summary

| Category | Count | Items |
|---|---|---|
| **OWNER APPROVED** | 3 | D03/R-008, D04/R-009, D05/R-010 |
| **CONFIRMED OBJECTIVE** | 1 | WIN-001 |
| **OPEN (blocking)** | 4 | D01, D02, D06, R-017 |
| **PROPOSED (confirm)** | 7 | D07, D08, D09, D10, D11, D12, D13, D14 |
| **Total requiring sign-off** | 11 | All of the above except D03, D04, D05 and WIN-001 |

**Rule freeze condition:** Only when every item above is signed does the ruleset become `DM-RULES-1.0` and Phase 2 is authorized.

---

**Worksheet prepared by:** Architect (Phase 1 review)  
**Date:** 2026-10-21
