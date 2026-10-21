# 10 — QA Test Matrix and Acceptance Criteria

**Document ID:** DM-QA-010  
**Version:** 0.1 DRAFT  
**Status:** Phase 1 — awaiting owner approval  
**Cross-references:** [03-canonical-ruleset-v1-draft](03-canonical-ruleset-v1-draft.md), [04-game-state-machine](04-game-state-machine.md), [06-api-and-realtime-contracts](06-api-and-realtime-contracts.md), [07-offline-online-parity-strategy](07-offline-online-parity-strategy.md), [08-security-privacy-and-compliance](08-security-privacy-and-compliance.md)

---

## Purpose

Define every category of testing required before MVP launch, with specific fixtures, pass criteria, and coverage requirements. This document is the single source of truth for QA scope.

## Scope

Covers rules engine tests, cross-language parity, client integration, network resilience, load testing, security testing, operations testing, platform testing, and simulation runs. Each section specifies what to test, expected result, and priority.

---

## 1. Rules Engine Test Matrix

### 1.1 Basic Legal Move Fixtures

| Fixture ID | Setup | Action | Expected Result | Rule Ref |
|---|---|---|---|---|
| RE-001 | Hand: [3♣, 7♣, A♥]; Lead: ♣ | Play 3♣ | Accepted; card moves to trick | R-005 |
| RE-002 | Hand: [3♣, 7♣, A♥]; Lead: ♣ | Play 7♣ | Accepted; card moves to trick | R-005 |
| RE-003 | Hand: [3♣, 7♣, A♥]; Lead: ♣ | Play A♥ | **Rejected**: must follow suit (holds ♣) | R-005 |
| RE-004 | Hand: [A♥, 3♦, 9♠]; Lead: ♣ | Play A♥ | Accepted; HIT (no ♣ in hand) | R-005 |
| RE-005 | Hand: [A♥, 3♦, 9♠]; Lead: ♣ | Play 3♦ | Accepted; HIT (no ♣ in hand) | R-005 |
| RE-006 | Hand: [A♥, 3♦, 9♠]; Lead: ♣ | Play 9♠ | Accepted; HIT (no ♣ in hand) | R-005 |
| RE-007 | Hand: [A♠]; only card | Play A♠ | Accepted; hand becomes empty | R-005 |
| RE-008 | Empty hand; player is active | Any play | **Rejected**: no cards (should be finished) | R-012 |

### 1.2 Round Resolution Fixtures

| Fixture ID | Trick Cards | Hit? | Winner | Cards Go To | Rule Ref |
|---|---|---|---|---|---|
| RR-001 | Lead: 8♣, Follow: 10♣, Q♣, 2♣ | No | Seat playing Q♣ (highest ♣) | Discard pile | R-007 |
| RR-002 | Lead: 8♣, Follow: J♣, 10♣, 2♥ | Yes | Seat playing J♣ (highest ♣) | Winner's hand | R-008 |
| RR-003 | Lead: 8♣, Follow: J♣, 10♣, A♥ | Yes | Seat playing J♣ (highest ♣, not A♥) | Winner's hand | R-008 [OWNER APPROVED 2026-10-21] |
| RR-004 | Lead: A♣, Follow: 2♣, 3♣, 4♣ | No | Seat playing A♣ (highest ♣) | Discard pile | R-006, R-007 |
| RR-005 | Lead: 2♣, Follow: 3♣, 5♥, 4♣ | Yes | Seat playing 4♣ (highest ♣) | Winner's hand | R-008 |
| RR-006 | Lead: K♠, Follow: A♠, 3♥, 7♠ | Yes | Seat playing A♠ (highest ♠) | Winner's hand | R-006, R-008 |
| RR-007 | Lead: 8♣, Follow: 8♣ duplicate | — | **ERROR**: duplicate card in trick | Invariant violation | INV-1 |

### 1.3 Finisher Determination Fixtures

| Fixture ID | Scenario | Expected Result | Rule Ref |
|---|---|---|---|
| FD-001 | Player plays last card; no hit; not winner | Player becomes finisher after discard resolution | R-009 [OWNER APPROVED 2026-10-21] |
| FD-002 | Player plays last card; hit; player is NOT winner | Winner collects cards → player's hand still empty → finisher | R-009 [OWNER APPROVED 2026-10-21] |
| FD-003 | Player plays last card; hit; player IS leading-suit winner | Winner collects cards → player now has cards → NOT a finisher | R-009 [OWNER APPROVED 2026-10-21], R-014 |
| FD-004 | Two players play last cards in same round; no hit | Both become finishers; rank by clockwise order from round leader (distinct consecutive positions, no ties) | R-010 [OWNER APPROVED 2026-10-21] |
| FD-005 | Two players play last cards; hit; winner is one of them | Winner gets cards back (not finisher); other player finishes | R-009 [OWNER APPROVED 2026-10-21], R-010 [OWNER APPROVED 2026-10-21] |
| FD-006 | All 4 players play last cards; no hit | **All finish simultaneously → DRAW / no Donkey** | R-013 [OPEN] |
| FD-007 | 3 players finished; 1 remains with cards | Remaining player declared Donkey | R-012 |
| FD-008 | 2 active players; both play last card; no hit | Both finish → DRAW / no Donkey (no one left with cards) | R-013 [OPEN] |

### 1.4 Turn Order and Leader Fixtures

| Fixture ID | Scenario | Expected Result | Rule Ref |
|---|---|---|---|
| TO-001 | A♠ holder in seat 2; match start | Seat 2 leads first round | R-003 |
| TO-002 | Round winner seat 3; next round | Seat 3 leads; turns proceed clockwise | R-004, R-007/R-008 |
| TO-003 | Round winner (seat 1) has no cards after resolution | Next leader: first active seat clockwise from seat 1 | R-011 |
| TO-004 | Seats 0,1,2,3; seat 1 finished | Round plays: seat 0, seat 2, seat 3 (seat 1 skipped) | R-004 |
| TO-005 | Only seats 0 and 3 active | Round plays: seat 0 and seat 3 only | R-004 |

### 1.5 Edge Case Fixtures

| Fixture ID | Scenario | Expected Result | Rule Ref |
|---|---|---|---|
| EC-001 | First round: A♠ holder leads any card (not A♠) | Accepted if R-003 approved as "any card" | R-003 [OPEN] |
| EC-002 | First round: A♠ holder forced to play A♠ | Accepted if R-003 approved as "must play A♠" | R-003 [OPEN] |
| EC-003 | Hit round: leading-suit winner exits after resolution | Next leader is clockwise from exited winner | R-011 |
| EC-004 | Match reaches 200 rounds or 30 minutes elapsed | Watchdog triggers on either condition; match voided, no Donkey awarded | R-017 [OPEN] |
| EC-005 | Player has 1 card; it's off-suit to lead | Player plays it (HIT); hand empty; collect goes to lead-suit winner; player finishes | R-009, R-010 |

---

## 2. State Invariant Tests (Property-Based)

These invariants must hold **at every state transition** and are tested via property-based/fuzz testing over randomized matches.

| Invariant ID | Property | Test Method |
|---|---|---|
| INV-001 | Exactly 52 distinct cards exist across all hands + current trick + discard pile | Assert at every transition in 100K simulation |
| INV-002 | No card appears in two locations simultaneously | Assert uniqueness across all containers |
| INV-003 | Hit resolution: trick cards move to exactly one winner hand; discard pile unchanged | Assert hand sizes and discard size after hit |
| INV-004 | No-hit resolution: trick cards move to discard; all hands unchanged | Assert hand sizes and discard size after no-hit |
| INV-005 | Current turn always belongs to an active (non-finished) seat | Assert at every WAITING_FOR_PLAY state |
| INV-006 | State version is strictly monotonically increasing | Assert version > previous version at every transition |
| INV-007 | Finish order is append-only; no finished player re-enters | Assert finished set only grows |
| INV-008 | Terminal result committed exactly once per match | Assert result event count = 1 at COMPLETED |
| INV-009 | No opponent hand data in any event payload for non-recipient seats | Assert seat-filtered output at every broadcast |
| INV-010 | actionId uniqueness: no two different state transitions share an actionId | Assert actionId unique constraint |

---

## 3. Mass Simulation Requirements

### 3.1 Parameters

| Parameter | Value |
|---|---|
| Minimum simulated matches | 100,000 |
| Bot policy | Random legal moves (uniform selection from legal set) |
| Shuffle | Seeded deterministic PRNG (not CSPRNG); seed recorded per match for reproducibility. Production uses CSPRNG — see doc 03 §2. |
| Timeout simulation | 5% of turns simulated as timeout → auto-play |
| Disconnect simulation | 2% of turns simulated as disconnect → bot takeover for remaining round |
| Invariant checks | All INV-001 through INV-010 at every state transition |

### 3.2 Pass Criteria

| Criterion | Threshold |
|---|---|
| Zero invariant violations | 0 out of 100K matches |
| Zero infinite loops | 0 matches exceeding watchdog threshold |
| All matches terminate | 100% reach COMPLETED, DRAW, or VOIDED |
| Seed reproducibility | Re-running any match with same seed produces identical event sequence |
| Round count distribution | Record histogram; flag outliers (>100 rounds) for manual review |
| Finisher distribution | Roughly uniform across seats (±5% at 100K scale) — no systematic bias |

### 3.3 Failure Handling

- Any invariant violation: emit match seed, full event log, and state at failure point.
- Auto-generate a deterministic test case from the failing seed.
- No simulation pass until all generated failure cases are resolved and added to the regression suite.

---

## 4. Cross-Language Golden Test Matrix

### 4.1 Fixture Format

Shared JSON fixtures consumed by both Dart and C# test harnesses:

```json
{
  "fixtureId": "GOLDEN-001",
  "rulesetVersion": "DM-RULES-1.0",
  "initialDeal": {
    "seat0": ["A♠", "K♠", "Q♠", ...],
    "seat1": [...],
    "seat2": [...],
    "seat3": [...]
  },
  "actions": [
    { "seat": 0, "card": "A♠", "actionId": "uuid-1" },
    { "seat": 1, "card": "K♣", "actionId": "uuid-2" },
    ...
  ],
  "expectedStates": [
    {
      "afterAction": "uuid-1",
      "stateVersion": 1,
      "trickCards": ["A♠"],
      "hands": { "seat0": [...], "seat1": [...], ... },
      "discardPile": [],
      "finishedSeats": []
    },
    ...
  ],
  "expectedResult": {
    "type": "COMPLETED",
    "positions": { "seat0": 1, "seat1": 2, "seat2": 3, "seat3": "DONKEY" }
  }
}
```

### 4.2 Required Golden Fixtures

| Fixture ID | Scenario | Purpose |
|---|---|---|
| GOLDEN-001 | Simple 4-round game, no hits | Basic flow + discard |
| GOLDEN-002 | Game with one hit in round 2 | Hit collection mechanics |
| GOLDEN-003 | Multiple hits across rounds | Extended hit handling |
| GOLDEN-004 | Player finishes mid-game; subsequent rounds skip them | Finisher mechanics |
| GOLDEN-005 | Hit winner's last card → collects cards back | R-009/R-014 edge case |
| GOLDEN-006 | Two players finish same round (if R-010 approved) | Simultaneous finisher |
| GOLDEN-007 | Winner exits → clockwise leader | R-011 leader replacement |
| GOLDEN-008 | All players empty simultaneously (if R-013 approved) | Draw scenario |
| GOLDEN-009 | Full game to completion with Donkey | Complete end-to-end |
| GOLDEN-010 | Game with timeout auto-plays | Timeout integration |

### 4.3 Parity Pass Criteria

- Dart engine and C# engine must produce **identical** `expectedStates` and `expectedResult` for every golden fixture.
- Comparison is on canonical JSON (sorted keys, normalized card IDs).
- Any divergence is a P0 blocker.

---

## 5. Client Integration Tests

### 5.1 Flutter Widget / Golden UI Tests

| Test ID | Scenario | Criteria |
|---|---|---|
| UI-001 | 13-card hand on 360px-wide phone | All cards visible; touch targets ≥24px exposed width |
| UI-002 | 13-card hand on 768px-wide tablet | Cards comfortably spaced; no overflow |
| UI-003 | Malayalam text in all UI elements | No truncation, no overflow, correct rendering |
| UI-004 | Text scale 2.0x with English | Layout intact; no overlap; scrollable where needed |
| UI-005 | Text scale 2.0x with Malayalam | Layout intact; wider text accommodated |
| UI-006 | Dark mode (system preference) | All colors use dark palette; text readable; cards distinct |
| UI-007 | Reduced motion enabled | All animations replaced with instant/fade; no motion sickness triggers |
| UI-008 | Safe area / notch handling | Content does not overlap notch or home indicator |
| UI-009 | Small phone (320px width) | Minimum layout functional; cards may scroll |
| UI-010 | Turn countdown at all stages | >10s normal, 5-10s warning, <5s urgent — visual states correct |

### 5.2 Interaction Tests

| Test ID | Scenario | Criteria |
|---|---|---|
| INT-001 | Tap legal card → card lifts + Play button appears | Correct visual feedback; Play button active |
| INT-002 | Tap illegal card | Card shakes; hint text shown; no selection |
| INT-003 | Tap selected card again | Card deselects; Play button deactivates |
| INT-004 | Tap Play button with selected card | Card animates to table; Play button deactivates |
| INT-005 | Tap Play button with no selection | Nothing happens |
| INT-006 | Select card A, then tap card B | Card A deselects; Card B selects |
| INT-007 | Drag card to table center (if enabled) | Card plays if legal; cancels if released outside zone |

---

## 6. Network and Contract Tests

### 6.1 REST API Tests

| Test ID | Endpoint | Scenario | Expected |
|---|---|---|---|
| API-001 | `POST /auth/guest` | Valid request | 200; access + refresh tokens |
| API-002 | `POST /auth/guest` | Rate limit exceeded | 429 |
| API-003 | `POST /auth/refresh` | Valid refresh token | 200; new token pair; old revoked |
| API-004 | `POST /auth/refresh` | Revoked refresh token | 401 |
| API-005 | `POST /auth/refresh` | Expired refresh token | 401 |
| API-006 | `POST /rooms` | Authenticated; valid request | 201; room with invite code |
| API-007 | `POST /rooms` | No auth header | 401 |
| API-008 | `POST /rooms/join` | Valid code; room not full | 200; seat assigned |
| API-009 | `POST /rooms/join` | Invalid code | 404 |
| API-010 | `POST /rooms/join` | Room full | 409 |
| API-011 | `POST /rooms/join` | Rate limit exceeded | 429 |
| API-012 | `GET /matches/{id}/snapshot` | Participant; match active | 200; own hand only |
| API-013 | `GET /matches/{id}/snapshot` | Non-participant | 403 |
| API-014 | `DELETE /account` | Authenticated | 202; deletion scheduled |

### 6.2 SignalR Hub Tests

| Test ID | Method | Scenario | Expected |
|---|---|---|---|
| HUB-001 | `PlayCard` | Valid play; correct turn | `ActionAcknowledged`; `CardPlayed` broadcast |
| HUB-002 | `PlayCard` | Card not in hand | `ActionRejected` with `CARD_NOT_IN_HAND` |
| HUB-003 | `PlayCard` | Not player's turn | `ActionRejected` with `NOT_YOUR_TURN` |
| HUB-004 | `PlayCard` | Stale `expectedStateVersion` | `ActionRejected` with `VERSION_MISMATCH` |
| HUB-005 | `PlayCard` | Exact duplicate `actionId` (same card, same seat) | Original `ActionAcknowledged` returned |
| HUB-005a | `PlayCard` | Same `actionId`, different card (same seat) | `ACTION_ID_CONFLICT`; original card not revealed |
| HUB-005b | `PlayCard` | Same `actionId` from different player | `ACTION_ID_CONFLICT`; original player's data not revealed |
| HUB-006 | `PlayCard` | After deadline | `ActionRejected` with `DEADLINE_PASSED` |
| HUB-007 | `PlayCard` | Must follow suit; plays off-suit | `ActionRejected` with `MUST_FOLLOW_SUIT` |
| HUB-008 | `JoinMatch` | Valid match + auth | `SeatAssigned` + current snapshot |
| HUB-009 | `JoinMatch` | Already in match (reconnect) | Resync snapshot; no duplicate seat |
| HUB-010 | `JoinMatch` | Not a participant | Error; no access |
| HUB-011 | `CardPlayed` broadcast | 4 clients connected | Each receives `CardPlayed` with only the played card; no hand data |
| HUB-012 | `RoundResolved` broadcast | Hit round | All receive trick cards + winner; no hand data |
| HUB-013 | `MatchCompleted` broadcast | Match ends | All receive positions; optionally full hand history |

---

## 7. Network Resilience Tests

| Test ID | Scenario | Duration | Expected |
|---|---|---|---|
| NET-001 | Simulated 2G latency (500ms+ RTT) | Full match | Match completes; UI shows countdown adjusted for latency |
| NET-002 | Simulated 3G latency (200ms RTT) | Full match | Match completes; smooth gameplay |
| NET-003 | 10% packet loss | Full match | Retries succeed; no missed events; match completes |
| NET-004 | Wi-Fi → mobile switch mid-match | — | Reconnect within grace period; match resumes |
| NET-005 | Airplane mode for 5 seconds | — | Disconnected state shown; reconnect on restore; snapshot resync |
| NET-006 | Airplane mode for 30 seconds | — | Bot takeover (if past grace period); reclaim on reconnect |
| NET-007 | App backgrounded for 10 seconds | — | Reconnect on foreground; snapshot resync |
| NET-008 | App force-killed and restarted | — | Rejoin match; snapshot resync from server |
| NET-009 | Server restart during active match | — | Match state recovered from journal; clients resync |
| NET-010 | Client sends 10 rapid duplicate `PlayCard` | — | Server processes first; returns idempotent acks for rest |
| NET-011 | Client sends actions out of order | — | Version mismatch rejection; client resyncs |

---

## 8. Load Tests

### 8.1 Progressive Load Tiers

| Tier | Concurrent Clients | Concurrent Matches | Duration | Metrics |
|---|---|---|---|---|
| L1 | 25 | 6 | 10 min | Baseline p50/p95 latency, CPU, RAM |
| L2 | 100 | 25 | 20 min | Same + connection resets, DB pool usage |
| L3 | 500 | 125 | 30 min | Same + error rate, GC pauses, DB IOPS |
| L4 | 1,000 | 250 | 60 min | Same + soak test: memory leaks, resource exhaustion |

### 8.2 Metrics to Record

| Metric | Acceptable (MVP) | Alert |
|---|---|---|
| `PlayCard` p50 latency | <50 ms | >100 ms |
| `PlayCard` p95 latency | <150 ms | >300 ms |
| SignalR broadcast latency | <100 ms p95 | >250 ms |
| Connection failure rate | <0.1% | >1% |
| Server CPU usage | <70% at L2 | >85% |
| Server RAM usage | <70% at L2 | >85% |
| Database connections | <80% pool | >90% pool |
| Match completion rate | >99% | <95% |
| Error rate (5xx) | <0.01% | >0.1% |

### 8.3 Stress Scenarios

| Test ID | Scenario | Purpose |
|---|---|---|
| STRESS-001 | 100 simultaneous room creates | Rate limiting works; no resource exhaustion |
| STRESS-002 | 50 clients all `PlayCard` at once (different matches) | Serialization per match; no cross-match interference |
| STRESS-003 | Slow client (5s response time) | Timeout fires; bot takes over; match continues |
| STRESS-004 | 1000 room code join attempts (brute force) | Rate limiting blocks; no code leak |
| STRESS-005 | Rapid connect/disconnect cycling | Connection handling stable; no memory leak |

---

## 9. Security Tests

| Test ID | Category | Scenario | Expected |
|---|---|---|---|
| SEC-001 | Hidden hand | Inspect all SignalR events as non-recipient | Zero opponent card IDs |
| SEC-002 | Hidden hand | Inspect REST snapshot as participant | Only own hand |
| SEC-003 | Hidden hand | Grep server logs during match | Zero hand arrays |
| SEC-004 | Hidden hand | Trigger crash; inspect report | Zero card data |
| SEC-005 | Auth | Use expired JWT | 401 on all endpoints |
| SEC-006 | Auth | Use valid JWT for wrong player's match | 403 |
| SEC-007 | Auth | Refresh with stolen token from different device | Reject or security event |
| SEC-008 | Input fuzz | Random bytes as `cardId` | Validation error; no crash |
| SEC-009 | Input fuzz | Random bytes as `matchId` | Validation error; no crash |
| SEC-010 | Input fuzz | Extremely long strings in all fields | Validation error; no buffer overflow |
| SEC-011 | Rate limit | Exceed room join limit | 429; subsequent attempts blocked |
| SEC-012 | Rate limit | Exceed auth endpoint limit | 429 |
| SEC-013 | Idempotency | Duplicate reward grant | Balance unchanged (idempotent) |
| SEC-014 | Idempotency | Duplicate `actionId` with different card (same seat) | `ACTION_ID_CONFLICT` returned; original card not revealed; anomaly logged |
| SEC-014a | Idempotency | Same `actionId` from different player's seat | `ACTION_ID_CONFLICT` returned; original player's action data not leaked |
| SEC-015 | RNG | 1M shuffle statistical distribution test | Chi-squared within expected variance |
| SEC-016 | Injection | SQL injection attempts in room code input | Parameterized queries; no injection |
| SEC-017 | Injection | XSS in player display name | Sanitized/escaped in all outputs |

---

## 10. Operations Tests

| Test ID | Scenario | Expected |
|---|---|---|
| OPS-001 | Full database backup and restore | Restored DB has all data; matches resumable |
| OPS-002 | Schema migration (forward) | Migration applies cleanly; no data loss |
| OPS-003 | Schema migration (rollback) | Previous version restored; no data loss |
| OPS-004 | Server rolling restart during matches | Matches recover from journal; clients resync |
| OPS-005 | Client with older ruleset version joins | Server rejects with `RULESET_VERSION_MISMATCH`; client prompted to update |
| OPS-006 | Health check endpoint | Returns 200 with service status |
| OPS-007 | Deployment rollback | Previous version deployed; matches in progress void or recover |
| OPS-008 | Log rotation at 7-day boundary | Old logs deleted; no PII retained beyond policy |
| OPS-009 | Account deletion after 30-day grace | Data anonymized per retention matrix |

---

## 11. Platform Tests

### 11.1 Device Matrix

| Category | Devices | Tests |
|---|---|---|
| Low-end Android | Redmi 9A or equivalent (2 GB RAM, HD screen) | Full match; ≥30 FPS; no crash |
| Mid-range Android | Samsung Galaxy A54 or equivalent | Full match; ≥55 FPS; all animations |
| High-end Android | Samsung Galaxy S24 or equivalent | Full match; 60 FPS; all effects |
| iPhone (older) | iPhone SE 3rd gen or equivalent | Full match; ≥55 FPS; safe areas |
| iPhone (current) | iPhone 15 or equivalent | Full match; 60 FPS; Dynamic Island safe area |
| Tablet | iPad Air or Samsung Tab A | Landscape/portrait adaptive layout |

### 11.2 Platform-Specific Tests

| Test ID | Platform | Scenario | Criteria |
|---|---|---|---|
| PLT-001 | Android | App in background → foreground | Reconnect; match resumes |
| PLT-002 | Android | Low memory pressure | App not killed during match |
| PLT-003 | Android | Split screen mode | UI scales; gameplay functional |
| PLT-004 | iOS | App in background → foreground | Reconnect; match resumes |
| PLT-005 | iOS | Incoming phone call during match | Match pauses (grace period); resumes after call |
| PLT-006 | iOS | Control Center opened during play | No crash; resumes correctly |
| PLT-007 | Both | System dark mode toggle during match | Theme updates; no crash |
| PLT-008 | Both | Text scale changed during match | Layout adjusts; no overlap |

---

## 12. MVP Entry / Exit Criteria

### 12.1 Entry Criteria (Start Implementation)

| Criterion | Owner | Status |
|---|---|---|
| Approved ruleset `DM-RULES-1.0` | Product owner | [PENDING] |
| All blocking decisions (D01–D12) signed | Product owner | [PENDING] |
| Approved API contract | Architect + owner | [PENDING] |
| Documented schema and migration plan | Architect | [PENDING] |
| Approved visual direction | Owner + designer | [PENDING] |
| CI pipeline targets documented (CI creation is a Phase 2 setup task) | Engineering | [DOCUMENTED] |

### 12.2 Exit Criteria (Ship MVP)

| Criterion | Measurement | Threshold |
|---|---|---|
| P0/P1 bugs | Count in issue tracker | 0 open |
| Rules engine tests | All fixtures passing | 100% |
| Mass simulation | 100K matches, zero invariant violations | 100K clean |
| Cross-language parity | All golden fixtures identical in Dart/C# | 100% match |
| Hidden-hand leak | Adversarial test suite | Zero leaks detected |
| End-to-end cross-device | Four human players complete matches | ≥3 successful matches |
| Reconnection | Disconnect/reconnect demonstrated | Match resumes correctly |
| Backup restore | Full restore demonstrated | Data intact |
| Store compliance | Privacy/deletion/disclosures reviewed | Checklist complete |
| Performance | Low-end device testing | ≥30 FPS; no crash |
| Launch rollback | Rollback tested | Previous version deployable |

---

## Open Questions

| ID | Question | Impact | Status |
|---|---|---|---|
| QA-Q1 | What is the target low-end Android device? | Defines performance floor | [OPEN] |
| QA-Q2 | Should simulation count be higher than 100K? | More confidence, more CI time | [PROPOSED: 100K] |
| QA-Q3 | Should cross-language parity include performance benchmarks or only state correctness? | State correctness sufficient for MVP | [PROPOSED] |
| QA-Q4 | Should load test L4 (1000 clients) be a launch gate or a post-launch target? | MVP may not need 1000 concurrent | [OPEN] |
| QA-Q5 | How many golden fixtures are sufficient? | 10 minimum; expandable | [PROPOSED: 10+] |
