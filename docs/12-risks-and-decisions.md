# 12 — Risk Register and Decision Log

**Document ID:** DM-DOC-12  
**Version:** 0.1 DRAFT  
**Date:** 2026-10-20  
**Status:** AWAITING OWNER APPROVAL — do not implement before sign-off

## Purpose

Consolidate every high-impact unresolved decision and identified risk for the Donkey multiplayer project. Each blocking decision must be explicitly approved before Phase 2 coding begins. The risk register captures technical, product, operational, and legal risks with likelihood, impact, and mitigations.

## Scope

Covers all items from the PRD draft decision register (D01–D12), plus additional risks identified during Phase 1 analysis. Cross-references rule IDs from `03-canonical-ruleset-v1-draft.md` and architecture from `05-architecture-and-data-model.md`.

---

## ID Cross-Reference

Three ID schemes are used across Phase 1 documents. This table maps between them:

| Decision (D-ID) | Rule (R-ID) | Ambiguity (HPA-ID) | Short Description | Status |
|---|---|---|---|---|
| D01 | R-002 | HPA-1 | Shuffle distribution policy | **[OWNER APPROVED 2026-10-21]** |
| D02 | R-003 | HPA-2 | A♠ opening constraint | **[OWNER APPROVED 2026-10-21]** |
| D03 | R-008 | HPA-3 | HIT winner — leading suit only or cross-suit | **[OWNER APPROVED 2026-10-21]** |
| D04 | R-009 | HPA-4 | Finisher timing — after resolution or immediate | **[OWNER APPROVED 2026-10-21]** |
| D05 | R-010 | HPA-5 | Simultaneous finisher ranking | **[OWNER APPROVED 2026-10-21]** |
| D06 | R-013 | HPA-6 | All players empty simultaneously | **[OWNER APPROVED 2026-10-21]** |
| D07 | — | HPA-7 | Timeout / disconnect / bot takeover policy | **[OWNER CONFIRMED 2026-10-21]** |
| D08 | — | HPA-8 | Bot fairness — no hidden hand access | **[OWNER CONFIRMED 2026-10-21]** |
| D09 | — | HPA-9 | Room start: 4 players only at MVP | **[OWNER CONFIRMED 2026-10-21]** |
| D10 | — | HPA-10 | Online bots: disclosed, queue fallback only | **[OWNER CONFIRMED 2026-10-21]** |
| D11 | — | HPA-11 | 2.5D rendering (see ADR-0001) | **[OWNER CONFIRMED 2026-10-21]** |
| D12 | — | HPA-12 | Dart SignalR client spike (see ADR-0002) | **[OWNER CONFIRMED 2026-10-21]** |
| D13 | — | — | Offline currency isolation | **[OWNER CONFIRMED 2026-10-21]** |
| D14 | — | — | India-first regional compliance | **[OWNER CONFIRMED 2026-10-21]** |
| — | R-017 | — | Watchdog threshold for pathological matches | [OPEN] |
| WIN-001 | — | — | Game objective: first to empty = winner, last with cards = Donkey | **[CONFIRMED 2026-10-21]** |

**Note on IDs:** D-IDs are used in decision workflows and the approval checklist. R-IDs are used in the canonical ruleset (doc 03). HPA-IDs are used in the rules evidence table (doc 02). All three refer to the same underlying decisions.

### Repository Source Files

The repository root contains two copies of the original PRD:

- `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT.md` — **Authoritative copy.** This is the original product/technical spec draft used as input to Phase 1 documentation.
- `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT-4.md` — **Duplicate.** Content-identical to the above. Appears to be a versioned copy. Should be removed or archived to avoid confusion. The `docs/` files supersede both copies as the canonical Phase 1 specification.

---

## 1. Decision Register

### BLOCKING GAMEPLAY DECISIONS

These must be signed before `DM-RULES-1.0` is issued.

| ID | Question | Recommended Default | Alternatives | Risk of Guessing Wrong | Evidence Needed | Owner Action |
|---|---|---|---|---|---|---|
| **D01** | **Shuffle distribution policy** — uniform CSPRNG 52-card shuffle and 13 each, or intentional suit-distribution skewing? | **APPROVED:** Uniform CSPRNG Fisher-Yates shuffle; 13 cards each; no suit-distribution skewing | ~~(a) Replicate competitor's undisclosed suit-skewing algorithm; (b) Implement a disclosed balancing heuristic (e.g., guarantee each player gets ≥2 of each suit)~~ | Resolved. Owner approved uniform shuffle for our original game. | Owner decision (2026-10-21). This is a product choice, not competitor verification. Competitor's distribution algorithm is proprietary and unpublished. | **OWNER APPROVED 2026-10-21.** Maps to R-002. |
| **D02** | **Must A♠ be the opening card?** The holder of A♠ starts — but must they lead A♠ specifically, or any card? | **APPROVED:** A♠ holder starts with any card; A♠ not mandatory | ~~(a) A♠ must be the first card played; (b) A♠ holder chooses but A♠ is highlighted as suggested~~ | Resolved. Owner approved free choice for our original game. | Owner decision (2026-10-21). "Starts" read as "goes first," not "must play a specific card." | **OWNER APPROVED 2026-10-21.** Maps to R-003. |
| **D03** | **HIT winner determination** — in a round with a hit, does the highest leading-suit card win, or the highest card across all suits? | **APPROVED:** Highest leading-suit card wins; off-suit cards (including higher ranks like A♥) cannot win the trick | ~~(a) Highest rank regardless of suit wins; (b) Off-suit cards ranked separately with a secondary comparison~~ | Resolved. Owner approved leading-suit-only rule for our original game. | Owner decision (2026-10-21). This is a product choice, not independent competitor verification. Published rules say "largest card" without specifying cross-suit. | **OWNER APPROVED 2026-10-21.** Maps to R-008. |
| **D04** | **Finisher timing** — does a player exit immediately on playing their last card, or after the round's collection/discard resolves? | **APPROVED:** After resolution; a hit collection may return cards to the player | ~~(a) Immediate exit on last card played (before resolution)~~ | Resolved. Owner approved after-resolution finisher timing. | Owner decision (2026-10-21). | **OWNER APPROVED 2026-10-21.** Maps to R-009. |
| **D05** | **Simultaneous finishers** — when multiple players reach zero cards in the same round, are they ranked or tied? | **APPROVED:** Deterministic clockwise podium order from the round leader's seat; distinct consecutive positions; no tied ranks | ~~(a) All simultaneous finishers share the same rank (tied); (b) Rank by who played first in the round~~ | Resolved. Owner approved clockwise deterministic ranking. | Owner decision (2026-10-21). | **OWNER APPROVED 2026-10-21.** Maps to R-010. |
| **D06** | **All players empty simultaneously** — what happens if every player runs out of cards in the same round? | **APPROVED:** Rank all qualifying finishers by R-010 clockwise order from the round leader; last-ranked finisher is Donkey | ~~(a) Technical draw / no Donkey; (b) Replay the match; (c) Void match~~ | Resolved. Owner approved R-010 clockwise ranking with Donkey assignment. | Owner decision (2026-10-21). Deterministic, consistent with R-010 mechanic. | **OWNER APPROVED 2026-10-21.** Maps to R-013. |
| **D07** | **Timeout and disconnect policy** | **CONFIRMED:** 20s action deadline, 15s disconnect grace, disclosed bot takeover, reclaim at round boundary | ~~(a) Longer/shorter timers; (b) Immediate forfeit on disconnect; (c) Pause match for all players; (d) No bot takeover — just run the clock~~ | Resolved. Owner confirmed recommended defaults. | Owner decision (2026-10-21). | **OWNER CONFIRMED 2026-10-21.** |
| **D08** | **Bot fairness** — can bots (offline or online) inspect hidden opponent hands? | **CONFIRMED:** No; bots see only their own hand and played cards (same as a human seat) | ~~(a) Allow offline bots to see all hands for stronger play; (b) Different difficulty levels have different visibility~~ | Resolved. Owner confirmed no hidden-hand access. | Owner decision (2026-10-21). | **OWNER CONFIRMED 2026-10-21.** |

### NON-BLOCKING BUT IMPORTANT DECISIONS

| ID | Question | Recommended Default | Alternatives | Risk of Guessing Wrong | Owner Action |
|---|---|---|---|---|---|
| **D09** | **Room start with unfilled seats** | **CONFIRMED:** 4 human or declared-bot seats; no 2/3-player variant at MVP | ~~(a) Allow 2-3 players with modified dealing; (b) Auto-fill with bots~~ | Resolved. Owner confirmed 4-player MVP. | **OWNER CONFIRMED 2026-10-21.** |
| **D10** | **Online bots vs real players** | **CONFIRMED:** Bots disclosed; only as queue fallback after user opt-in or timeout | ~~(a) Silent bot fill; (b) No online bots ever~~ | Resolved. Owner confirmed disclosure policy. | **OWNER CONFIRMED 2026-10-21.** |
| **D11** | **2.5D vs real 3D rendering** | **CONFIRMED:** Flame 2.5D with authored assets; no Unity/Unreal | ~~(a) Unity with Flutter embedding; (b) Raw Flutter Canvas only~~ | Resolved. See ADR-0001. | **OWNER CONFIRMED 2026-10-21.** |
| **D12** | **Dart SignalR client package** | **CONFIRMED:** Compatibility spike against .NET 10 in Phase 4; fallback to raw WebSocket + custom protocol | ~~(a) Use `signalr_netcore` assuming compatibility; (b) Build custom WebSocket layer from day one~~ | Resolved. See ADR-0002. | **OWNER CONFIRMED 2026-10-21.** |
| **D13** | **Offline currency isolation** | **CONFIRMED:** Offline coins/progression fully separate from online authoritative balance | ~~(a) Allow offline coins to sync to online on connection; (b) No offline progression at all~~ | Resolved. Owner confirmed isolation. | **OWNER CONFIRMED 2026-10-21.** |
| **D14** | **Regional compliance** | **CONFIRMED:** India launch with non-gambling economy; legal review before international expansion | ~~(a) Launch globally immediately; (b) Restrict to specific states~~ | Resolved. Owner confirmed India-first + legal review. | **OWNER CONFIRMED 2026-10-21.** |

---

## 2. Risk Register

### 2.1 Risk Matrix

**Likelihood:** L=Low, M=Medium, H=High  
**Impact:** 1=Minor, 2=Moderate, 3=Major, 4=Critical

| Risk ID | Category | Description | Likelihood | Impact | Score | Mitigation |
|---|---|---|---|---|---|---|
| R-TECH-01 | Technical | Dart SignalR client incompatible with .NET 10 SignalR hub | M | 3 | 6 | Early compatibility spike in Phase 4; fallback to raw WebSocket + JSON protocol. See ADR-0002. |
| R-TECH-02 | Technical | Flame performance insufficient on low-end Android devices (2GB RAM, Mali GPU) | M | 3 | 6 | Quality presets (low/med/high); static table fallback; texture atlas budget caps; device testing matrix. See ADR-0001. |
| R-TECH-03 | Technical | .NET 10 LTS ecosystem immaturity or breaking changes in early months | L | 2 | 2 | .NET 10 is current LTS as of Oct 2026 with standard 3-year support. Pin SDK version; follow release notes. |
| R-TECH-04 | Technical | Cross-language rule engine divergence (Dart vs C#) | M | 4 | 8 | Shared JSON golden fixtures, CI parity gate, ruleset version field on every match, integer-only logic. See ADR-0003. |
| R-TECH-05 | Technical | PostgreSQL event journal grows unbounded for long-running matches or high volume | L | 2 | 2 | Snapshot compaction strategy; matches are short-lived (minutes); archive completed match events after retention period. |
| R-PROD-01 | Product | Insufficient differentiation from Donkey Master competitor | M | 3 | 6 | Original visual identity, premium feel, Malayalam localization, offline-first experience, transparent multiplayer. |
| R-PROD-02 | Product | Low initial user acquisition / empty online queues | H | 3 | 9 | Offline mode as primary entry; bot fill with disclosure for queue fallback; private rooms for friend groups; phased rollout starting with Kerala community. |
| R-PROD-03 | Product | Rule ambiguities resolved differently from player expectations | M | 2 | 4 | Competitor gameplay observation for D02/D03/D08; in-app rules glossary; willingness to patch rules post-launch with versioning. |
| R-OPS-01 | Operational | Solo developer bottleneck across Flutter, ASP.NET, DevOps, art, QA | H | 3 | 9 | Phase deliverables scoped to sequential milestones; no parallel workstreams assumed; outsource art assets; accept longer timeline. |
| R-OPS-02 | Operational | Insufficient real-device test coverage (especially iOS) | M | 2 | 4 | Minimum: 1 low-end Android, 1 mid Android, 1 current iPhone; CI emulators for regression; beta testers expand coverage. |
| R-OPS-03 | Operational | Backend single-point-of-failure on single VPS | M | 3 | 6 | Daily automated backups; tested restore procedure; graceful match void on irrecoverable state; Docker Compose restart policy. |
| R-LEGAL-01 | Legal | Simulated gambling classification in Indian states | L | 4 | 4 | No real money, no cash prizes, no transferable currency. Legal review before launch. Conservative economy design. |
| R-LEGAL-02 | Legal | App store rejection for gambling-adjacent content | L | 3 | 3 | Accurate age rating; no real-money mechanics; clear store listing description; comply with Apple/Google ad policies. |
| R-LEGAL-03 | Legal | Privacy regulation compliance (India DPDP Act, GDPR for international) | M | 3 | 6 | Minimal data collection; guest accounts; account deletion flow; privacy policy; consent for analytics/ads; see `08-security-privacy-and-compliance.md`. |
| R-SEC-01 | Security | Hidden hand data leaked via WebSocket events, error messages, or logs | M | 4 | 8 | Seat-filtered DTOs; adversarial payload inspection tests; no card data in error messages; sanitized logging. See `08-security-privacy-and-compliance.md`. |
| R-SEC-02 | Security | Replay/duplicate action exploits (double card play, double reward) | M | 4 | 8 | UUID `actionId` with unique constraint; `expectedStateVersion` optimistic concurrency; idempotent acknowledgment. |

### 2.2 Top Risks by Score

| Rank | Risk | Score | Primary Mitigation |
|---|---|---|---|
| 1 | R-PROD-02: Empty online queues | 9 | Offline-first design; bot queue fallback |
| 2 | R-OPS-01: Solo developer bottleneck | 9 | Sequential phases; outsource art; longer timeline |
| 3 | R-TECH-04: Rule engine divergence | 8 | CI parity gate; shared fixtures |
| 4 | R-SEC-01: Hidden hand leakage | 8 | Seat-filtered DTOs; adversarial testing |
| 5 | R-SEC-02: Replay/duplicate exploits | 8 | Idempotent actions; version checks |
| 6 | R-TECH-01: SignalR Dart compatibility | 6 | Early spike; WebSocket fallback |
| 7 | R-TECH-02: Flame on low-end devices | 6 | Quality presets; static fallback |
| 8 | R-PROD-01: Competitor differentiation | 6 | Original art; premium UX; localization |
| 9 | R-OPS-03: Backend SPOF | 6 | Backups; restore drills; void policy |
| 10 | R-LEGAL-03: Privacy compliance | 6 | Minimal data; deletion flow; consent |

---

## 3. Decision Dependencies

Some decisions are interlinked and must be resolved together:

```
D01 (shuffle) ──── independent; OWNER APPROVED 2026-10-21 (uniform CSPRNG)
D02 (A♠ opening) ── independent; OWNER APPROVED 2026-10-21 (free choice)
D03 (HIT winner) ── CRITICAL; affects D04, D05, D06 resolution logic
     │
     ├── D04 (finisher timing) ── affects D05, D06
     │    │
     │    ├── D05 (simultaneous finishers)
     │    └── D06 (all empty) ── OWNER APPROVED 2026-10-21
     │
     └── D07 (timeout/disconnect) ── OWNER CONFIRMED 2026-10-21
          │
          ├── D08 (bot fairness) ── OWNER CONFIRMED 2026-10-21
          └── D10 (online bot disclosure) ── OWNER CONFIRMED 2026-10-21

D09 (4-player only) ── OWNER CONFIRMED 2026-10-21
D11 (2.5D) ──── OWNER CONFIRMED 2026-10-21; approved by ADR-0001
D12 (SignalR client) ── OWNER CONFIRMED 2026-10-21; resolved by Phase 4 spike
D13 (offline currency) ── OWNER CONFIRMED 2026-10-21
D14 (regional compliance) ── OWNER CONFIRMED 2026-10-21; legal review parallel to development
```

**Critical path:** ~~D03~~ → ~~D04~~ → ~~D05~~/~~D06~~ are all resolved. D03, D04, D05, and D06 are now OWNER APPROVED (2026-10-21). The full gameplay resolution chain is complete.

---

## 4. Decision Workflow

### Who Approves

- **Product owner** (project stakeholder): all gameplay rule decisions (D01–D06), UX decisions (D07–D10), scope decisions (D09, D14)
- **Technical lead** (may be same person for solo developer): architecture decisions (D11–D12), confirmed by ADRs
- **Legal advisor** (external if needed): D14 regional compliance

### What Blocks Phase 2

Phase 2 (deterministic engine implementation) **cannot begin** until:

1. ~~D01 (shuffle policy) is approved → determines shuffle algorithm~~ **OWNER APPROVED 2026-10-21**
2. ~~D02 (A♠ opening) is approved → determines first-turn logic~~ **OWNER APPROVED 2026-10-21**
3. ~~D03 (HIT winner) is approved → determines round resolution~~ **OWNER APPROVED 2026-10-21**
4. ~~D04 (finisher timing) is approved → determines exit logic~~ **OWNER APPROVED 2026-10-21**
5. ~~D05 (simultaneous finishers) is approved → determines ranking logic~~ **OWNER APPROVED 2026-10-21**
6. ~~D06 (all empty) is approved → determines terminal condition~~ **OWNER APPROVED 2026-10-21**
7. ~~D07 (timeout/disconnect) is at least provisionally approved → affects bot integration~~ **OWNER CONFIRMED 2026-10-21**
8. ~~D08 (bot fairness) is approved → determines bot decision API~~ **OWNER CONFIRMED 2026-10-21**

**Approval artifact:** Signed `RULE_FREEZE_APPROVAL_CHECKLIST.md` with ruleset version `DM-RULES-1.0` and date.

### What Can Proceed in Parallel

- D11 (2.5D): ADR-0001 provides recommendation; art direction can begin
- D12 (SignalR): spike scheduled for Phase 4; does not block engine work
- D13 (offline currency): isolation is the safe default; economy design is post-MVP
- D14 (regional compliance): legal review can proceed alongside development

---

## 5. Assumptions

1. Solo developer with Flutter and C# experience; no dedicated QA, DevOps, or artist.
2. Art assets will be outsourced or created with rendering tools; timeline depends on vendor.
3. Single VPS deployment is sufficient for initial launch scale.
4. Legal review for Indian market is a parallel workstream, not a Phase 1 blocker.
5. No competitor code, assets, or protocols will be examined or reverse-engineered.

## Open Questions

- Should a formal competitor gameplay observation session be conducted to resolve D02, D03, D08? If so, who conducts it and how are findings documented?
- Is there budget for external art asset creation? Timeline impact?
- Is legal counsel available for D14 or is this self-assessed?

## Acceptance Criteria

- [ ] Every D01–D14 decision has an explicit APPROVED / REJECTED / DEFERRED status with date
- [ ] All BLOCKING decisions (D01–D08) resolved before Phase 2 start
- [ ] Risk register reviewed and accepted; top-5 mitigations have owners
- [ ] `RULE_FREEZE_APPROVAL_CHECKLIST.md` signed with `DM-RULES-1.0` version

## Cross-References

- `02-verified-rules-and-open-questions.md` — source evidence for each rule
- `03-canonical-ruleset-v1-draft.md` — the ruleset these decisions freeze
- `ADR/0001-flutter-flame-2-5d.md` — D11 technical basis
- `ADR/0002-aspnet-core-signalr-and-durable-actors.md` — D12 technical basis
- `ADR/0003-two-language-rules-parity.md` — R-TECH-04 mitigation
- `08-security-privacy-and-compliance.md` — R-SEC-01, R-SEC-02, R-LEGAL-03 details
- `PHASE_1_REVIEW_REPORT.md` — consolidated review and approval request
