# ADR-0003: Two-Language Rules Engine Parity (Dart + C#)

**Status:** PROPOSED  
**Date:** 2026-10-20  
**Decision Makers:** Product Owner / Technical Lead  
**Relates to:** R-016, `07-offline-online-parity-strategy.md`, `10-qa-test-matrix-and-acceptance.md`

## Context

The Donkey card game must work fully offline (no network required) on mobile devices AND online with server-authoritative state. This creates a fundamental architectural requirement:

- **Offline play** runs on the Flutter client → requires a **Dart** rules engine
- **Online play** is server-authoritative on ASP.NET Core → requires a **C#** rules engine

Both engines must produce identical results for identical inputs. Any divergence means a legal offline move could be rejected online (or vice versa), destroying player trust and creating exploitable inconsistencies.

The alternative — running only one engine — was considered and rejected:

- **Dart-only (client-authoritative online):** Unacceptable. Server must validate all moves; client cannot be trusted for online play. A cheating client could play illegal cards, fabricate outcomes, or manipulate state.
- **C#-only (server-required for offline):** Defeats the offline-first requirement. Users must play without any network connection.
- **Shared language via transpilation or WASM:** Dart-to-C# or C#-to-Dart transpilers are immature or nonexistent for this use case. Running C# via WASM on mobile adds complexity and bundle size disproportionate to the game logic size.

## Decision

**Maintain two independent implementations of the rules engine — one in Dart, one in C# — with parity enforced by a shared golden fixture suite and CI gates.**

### Implementation Constraints

Both engines must follow these constraints to ensure parity:

1. **Integer-only arithmetic:** All card rank comparisons, hand sizes, and sequence numbers use integers. No floating-point operations in game logic. This eliminates platform-specific floating-point behavior.

2. **No platform RNG in test fixtures:** Test fixtures use **fixed, pre-determined card decks** (the deck order is specified in the fixture JSON). Shuffle/RNG is tested separately for statistical properties; golden parity tests never compare two different RNG implementations.

3. **Deterministic input → deterministic output:** Given the same initial deck order and the same sequence of player actions, both engines must produce identical state at every step. No randomness in resolution logic.

4. **Ruleset version field:** Every match (online and offline) carries a `rulesetVersion` string (e.g., `"DM-RULES-1.0"`). The version determines which rules apply. Version mismatches between client and server are rejected at match join time.

5. **Canonical state representation:** Both engines serialize state to a canonical JSON format for comparison. The schema is defined once and shared.

### Golden Fixture Format

```json
{
  "fixtureId": "GF-001-no-hit-basic",
  "rulesetVersion": "DM-RULES-1.0",
  "description": "Basic 4-player round, no hit, Q♣ wins",
  "initialState": {
    "deck": ["8C", "10C", "QC", "2C", "...all 52 cards in deal order..."],
    "seats": [
      {"seatIndex": 0, "hand": ["8C", "..."], "status": "ACTIVE"},
      {"seatIndex": 1, "hand": ["10C", "..."], "status": "ACTIVE"},
      {"seatIndex": 2, "hand": ["QC", "..."], "status": "ACTIVE"},
      {"seatIndex": 3, "hand": ["2C", "..."], "status": "ACTIVE"}
    ],
    "currentLeader": 0,
    "discardPile": [],
    "stateVersion": 1
  },
  "actions": [
    {"seat": 0, "card": "8C", "expectedVersion": 1},
    {"seat": 1, "card": "10C", "expectedVersion": 2},
    {"seat": 2, "card": "QC", "expectedVersion": 3},
    {"seat": 3, "card": "2C", "expectedVersion": 4}
  ],
  "expectedResults": [
    {
      "afterAction": 0,
      "stateVersion": 2,
      "currentTrick": ["8C"],
      "leadSuit": "C",
      "nextSeat": 1
    },
    {
      "afterAction": 3,
      "stateVersion": 5,
      "roundResult": "NO_HIT",
      "roundWinner": 2,
      "trickCards": ["8C", "10C", "QC", "2C"],
      "trickDestination": "DISCARD",
      "nextLeader": 2
    }
  ]
}
```

Each fixture specifies:
- A fixed initial state (no RNG)
- A sequence of player actions
- The expected state after each action or round resolution
- The expected outcome (round winner, trick destination, finisher status, etc.)

### Fixture Categories

| Category | Count (minimum) | Purpose |
|---|---|---|
| No-hit basic | 5+ | Simple rounds, various winning ranks |
| Hit basic | 5+ | Single hit, leading-suit winner takes all |
| Off-suit high rank hit | 3+ | Verify off-suit A cannot beat leading-suit J (per approved R-008) |
| Follow-suit enforcement | 5+ | Legal/illegal move validation |
| Finisher after resolution | 3+ | Player empties hand; verify exit timing (R-009) |
| Simultaneous finishers | 3+ | Multiple players empty in same round (R-010) |
| All empty draw | 2+ | Every player empties simultaneously (R-013) |
| Last player Donkey | 3+ | Terminal condition detection |
| Leader exits, clockwise skip | 3+ | Next leader selection when winner finishes |
| Hit returns cards to empty player | 2+ | Collection restores hand; player does not exit (R-014) |
| Two active seats | 2+ | End-game with two players |
| Full match simulation | 5+ | Complete games from deal to Donkey/draw |
| Illegal move rejection | 5+ | Out-of-turn, wrong suit, nonexistent card |
| Idempotent action replay | 3+ | Same actionId replayed returns same ack |

**Target: 50+ fixtures at launch, growing to 100+ as edge cases are discovered.**

### CI Validation Pipeline

```
1. Generate/update fixtures from canonical fixture repository (JSON files)
2. Run Dart test suite: load fixture → execute actions → compare state → PASS/FAIL
3. Run C# test suite: load same fixture → execute actions → compare state → PASS/FAIL
4. Compare: if both PASS with identical canonical state hashes → GREEN
5. Any mismatch → RED, block merge
```

Both test suites read the same fixture files. The fixtures are the shared contract.

## Ruleset Versioning

### Version Format

`DM-RULES-{major}.{minor}` — e.g., `DM-RULES-1.0`

- **Major:** Breaking rule changes (e.g., changing HIT winner logic). Requires new fixtures. Old and new versions may coexist during migration.
- **Minor:** Clarifications or edge-case additions that do not change behavior for existing fixtures.

### Version Enforcement

- Every match record stores `rulesetVersion`
- Online: server checks client's supported ruleset version at match join; rejects outdated clients with an upgrade prompt
- Offline: saved games store `rulesetVersion`; engine loads appropriate version logic
- Fixtures are tagged with `rulesetVersion`; CI runs each fixture against its corresponding engine version

### Version Mismatch Handling

| Scenario | Behavior |
|---|---|
| Client version < server version (minor) | Server accepts; minor versions are backward-compatible |
| Client version < server version (major) | Server rejects join; client shows "Update required" |
| Client version > server version | Server rejects join (client is ahead of server — unusual but possible during rollout) |
| Saved offline game with old version | Load with old version engine; offer to start new game with current version |

## Consequences

### Positive

- Offline play works without any network dependency
- Server remains authoritative for online — no client trust required
- Shared fixtures provide a clear, testable contract
- Any divergence is caught in CI before reaching users
- Ruleset versioning supports future rule changes without breaking existing matches
- Integer-only logic eliminates platform-specific arithmetic issues

### Negative

- Two implementations to maintain for every rule change — development cost doubles for game logic
- Risk of subtle divergence in edge cases not covered by fixtures
- Fixture suite must be comprehensive — a missing fixture is a missing guarantee
- Both engines must be updated and released together for major version changes

### Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Subtle divergence in uncovered edge case | Medium | High | Property-based fuzz testing with random legal moves (100,000+ simulated matches); expand fixture suite continuously |
| Developer forgets to update both engines | Medium | High | CI gate: no merge without both test suites green; checklist in PR template |
| Fixture format becomes complex/hard to maintain | Low | Medium | Fixture generator tool (can be a simple script); fixtures are JSON, human-readable |
| Major rule change requires large fixture rewrite | Low | Medium | Versioned fixtures; old fixtures kept for old version; new version gets new fixture set |
| Platform-specific integer overflow | Very Low | High | Dart and C# both use 64-bit integers by default; card game values never approach limits; add assertion guards |

## Alternatives Considered

| Alternative | Reason Not Chosen |
|---|---|
| **Single Dart engine (client-authoritative online)** | Unacceptable security posture; server cannot trust client |
| **Single C# engine (server-required for offline)** | Defeats offline-first requirement |
| **C# to Dart transpilation** | No mature, production-ready transpiler exists |
| **Shared logic via WASM** | Adds significant bundle size and complexity for relatively simple game logic (~500–1000 LOC per engine) |
| **Protocol Buffers / IDL-generated logic** | Game rules are behavioral logic, not data serialization; IDL cannot express validation and resolution semantics |
| **Shared Rust core with FFI** | Cross-compilation to iOS/Android adds build complexity; Dart FFI works but adds a native dependency; overkill for the logic volume |

## References

- `07-offline-online-parity-strategy.md` — strategy details and CI pipeline design
- `03-canonical-ruleset-v1-draft.md` — the rules both engines implement
- `10-qa-test-matrix-and-acceptance.md` — test categories and acceptance criteria
- `04-game-state-machine.md` — state transitions both engines must match
