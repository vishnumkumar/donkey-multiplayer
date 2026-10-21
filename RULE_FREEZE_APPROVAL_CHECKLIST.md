# Donkey Game — Phase 1 Rule Freeze Checklist

Status: **PENDING OWNER APPROVAL**. Check and sign every blocking point before Phase 2 coding.

### Blocking Gameplay Decisions

- [x] D01 (R-002) — Shuffle policy: uniform CSPRNG, no suit skewing. **[OWNER APPROVED 2026-10-21]** — product owner's choice for our original game; no suit-distribution skewing or balancing heuristic.
- [x] D02 (R-003) — A♠ holder may lead any card (free choice). **[OWNER APPROVED 2026-10-21]** — product owner's choice; "starts" means goes first, A♠ not mandatory.
- [x] D03 (R-008) — HIT winner: highest leading-suit card only. **[OWNER APPROVED 2026-10-21]** — product owner's choice for our original game, not independent competitor verification.
- [x] D04 (R-009) — Finisher timing: after round resolution. **[OWNER APPROVED 2026-10-21]** — finisher identified only after full round settlement including HIT collection.
- [x] D05 (R-010) — Simultaneous finishers: distinct clockwise positions from round leader. **[OWNER APPROVED 2026-10-21]** — no tied ranks; deterministic ordering.
- [x] D06 (R-013) — All players empty simultaneously: rank by R-010 clockwise order; last-ranked finisher is Donkey. **[OWNER APPROVED 2026-10-21]** — no draw; deterministic ranking consistent with R-010.
- [x] D07 — Timeout/disconnect policy: 20s deadline, 15s grace, bot takeover, reclaim at round boundary. **[OWNER CONFIRMED 2026-10-21]** — recommended defaults accepted.
- [x] D08 — Bot fairness: bots see only own hand + played cards. **[OWNER CONFIRMED 2026-10-21]** — recommended default accepted; no hidden-hand access.
- [x] R-017 — Watchdog: void after 200 rounds or 30 minutes, whichever comes first; never assign arbitrary Donkey. **[OWNER APPROVED 2026-10-21]**

### Confirmed Product Objectives

- [x] **WIN-001** — First qualifying finisher (empty hand after round settlement per R-009) takes first place = game winner. Last player with cards = Donkey. **[CONFIRMED 2026-10-21]** — "Successfully" now defined by R-009 (APPROVED). Simultaneous finisher ranking now resolved by R-010/D05 (OWNER APPROVED 2026-10-21). All-empty resolved by R-013/D06 (OWNER APPROVED 2026-10-21).

### Non-Blocking but Important Decisions

- [x] D09 — MVP 4 players only; no 2/3-player variant until separately specified. **[OWNER CONFIRMED 2026-10-21]**
- [x] D10 — Online bots: disclosed, queue fallback only; all bot-controlled seats labeled. **[OWNER CONFIRMED 2026-10-21]**
- [x] D11 — 2.5D rendering accepted as visual direction (see ADR-0001). **[OWNER CONFIRMED 2026-10-21]**
- [x] D12 — Dart SignalR client compatibility spike in Phase 4; WebSocket fallback ready (see ADR-0002). **[OWNER CONFIRMED 2026-10-21]**
- [x] D13 — Offline virtual coins stay separate from online server wallet. **[OWNER CONFIRMED 2026-10-21]**
- [x] D14 — India-first launch; no wagering/cash prizes; legal review before international. **[OWNER CONFIRMED 2026-10-21]**

### Cross-References

All decision IDs (D01–D14) map to rule IDs (R-xxx) and ambiguity IDs (HPA-xxx) in `docs/12-risks-and-decisions.md` § ID Cross-Reference table.

**Approval fields**

Approved ruleset version: ____________________

Owner / reviewer: ____________________________

Date: _______________________________________

Unresolved exceptions and assigned action: __________________________

**Rule freeze condition:** Only when all blocking gameplay decisions are signed, publish version `DM-RULES-1.0` and authorize Phase 2. Until then, all documents remain *DRAFT*.
