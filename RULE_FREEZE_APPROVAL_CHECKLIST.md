# Donkey Game — Phase 1 Rule Freeze Checklist

Status: **PENDING OWNER APPROVAL**. Check and sign every blocking point before Phase 2 coding.

### Blocking Gameplay Decisions

- [ ] D01 (R-002) — Shuffle policy: uniform CSPRNG, no suit skewing **(recommended)** or intentional suit-distribution policy? **[OPEN]**
- [ ] D02 (R-003) — A♠ holder may lead any card **(recommended)** or must lead A♠? **[OPEN]**
- [x] D03 (R-008) — HIT winner: highest leading-suit card only. **[OWNER APPROVED 2026-10-21]** — product owner's choice for our original game, not independent competitor verification.
- [ ] D04 (R-009) — Finisher timing: after round resolution **(recommended)** or immediate on last card played?
- [ ] D05 (R-010) — Simultaneous finishers: clockwise podium order **(proposed)** or tied ranks? **[OPEN]**
- [ ] D06 (R-013) — All players empty simultaneously: draw / no Donkey **(recommended)** or replay? **[OPEN]**
- [ ] D07 — Timeout/disconnect policy: 20s deadline, 15s grace, bot takeover, reclaim at round boundary **(proposed)** or alternative timers?
- [ ] D08 — Bot fairness: bots see only own hand + played cards **(recommended)** or allow hidden-hand access for difficulty?
- [ ] R-017 — Watchdog: void after 200 rounds or 30 minutes **(proposed)** or alternative threshold? **[OPEN]**

### Confirmed Product Objectives

- [x] **WIN-001** — First player to successfully empty their entire hand = first-place game winner. Last player with cards = Donkey. **[CONFIRMED 2026-10-21]** — Precise mechanics ("successfully") depend on R-009 finisher timing; simultaneous finisher ranking (R-010/D05), all-empty draw (R-013/D06) not automatically approved by this statement.

### Non-Blocking but Important Decisions

- [ ] D09 — MVP 4 players only; no 2/3-player variant until separately specified.
- [ ] D10 — Online bots: disclosed, queue fallback only; all bot-controlled seats labeled.
- [ ] D11 — 2.5D rendering accepted as visual direction (see ADR-0001).
- [ ] D12 — Dart SignalR client compatibility spike in Phase 4; WebSocket fallback ready (see ADR-0002).
- [ ] D13 — Offline virtual coins stay separate from online server wallet.
- [ ] D14 — India-first launch; no wagering/cash prizes; legal review before international.

### Cross-References

All decision IDs (D01–D14) map to rule IDs (R-xxx) and ambiguity IDs (HPA-xxx) in `docs/12-risks-and-decisions.md` § ID Cross-Reference table.

**Approval fields**

Approved ruleset version: ____________________

Owner / reviewer: ____________________________

Date: _______________________________________

Unresolved exceptions and assigned action: __________________________

**Rule freeze condition:** Only when all blocking gameplay decisions are signed, publish version `DM-RULES-1.0` and authorize Phase 2. Until then, all documents remain *DRAFT*.
