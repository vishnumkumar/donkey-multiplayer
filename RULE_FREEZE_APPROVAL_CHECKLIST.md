# Donkey Game — Phase 1 Rule Freeze Checklist

Status: **PENDING OWNER APPROVAL**. Check and sign every blocking point before Phase 2 coding.

### Blocking Gameplay Decisions

- [x] D01 (R-002) — Shuffle policy: uniform CSPRNG, no suit skewing. **[OWNER APPROVED 2026-10-21]** — product owner's choice for our original game; no suit-distribution skewing or balancing heuristic.
- [x] D02 (R-003) — A♠ holder may lead any card (free choice). **[OWNER APPROVED 2026-10-21]** — product owner's choice; "starts" means goes first, A♠ not mandatory.
- [x] D03 (R-008) — HIT winner: highest leading-suit card only. **[OWNER APPROVED 2026-10-21]** — product owner's choice for our original game, not independent competitor verification.
- [x] D04 (R-009) — Finisher timing: after round resolution. **[OWNER APPROVED 2026-10-21]** — finisher identified only after full round settlement including HIT collection.
- [x] D05 (R-010) — Simultaneous finishers: distinct clockwise positions from round leader. **[OWNER APPROVED 2026-10-21]** — no tied ranks; deterministic ordering.
- [x] D06 (R-013) — All players empty simultaneously: rank by R-010 clockwise order; last-ranked finisher is Donkey. **[OWNER APPROVED 2026-10-21]** — no draw; deterministic ranking consistent with R-010.
- [ ] D07 — Timeout/disconnect policy: 20s deadline, 15s grace, bot takeover, reclaim at round boundary **(proposed)** or alternative timers?
- [ ] D08 — Bot fairness: bots see only own hand + played cards **(recommended)** or allow hidden-hand access for difficulty?
- [ ] R-017 — Watchdog: void after 200 rounds or 30 minutes **(proposed)** or alternative threshold? **[OPEN]**

### Confirmed Product Objectives

- [x] **WIN-001** — First qualifying finisher (empty hand after round settlement per R-009) takes first place = game winner. Last player with cards = Donkey. **[CONFIRMED 2026-10-21]** — "Successfully" now defined by R-009 (APPROVED). Simultaneous finisher ranking now resolved by R-010/D05 (OWNER APPROVED 2026-10-21). All-empty draw (R-013/D06) remains OPEN — not automatically approved.

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
