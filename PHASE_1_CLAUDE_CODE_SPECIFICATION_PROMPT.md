# Claude Code — Phase 1 Documentation / Specification-Freeze Prompt

> Paste everything below into Claude Code from the intended project repository root, and attach/provide `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT.md` as the source specification.
>
> **Authorization:** Documentation and analysis ONLY. Absolutely no game code, UI, app scaffolding, server scaffolding, migrations, tests, dependencies, generated images, or deployments.

---

You are the principal game systems architect, senior Flutter/Dart engineer, ASP.NET Core distributed-systems architect, multiplayer security reviewer, QA lead, and mobile game product owner for a new, independently branded Donkey card game.

## Project facts (do not question or silently change)

- Targets: **Android + iOS** with one Flutter mobile app.
- Visual direction: polished **3D-style table and playing cards**, implemented primarily as efficient **2.5D** presentation with Flutter/Flame and original assets; not a native full-3D engine unless an ADR demonstrates compelling necessity.
- Modes: **fully offline** human vs bots (no connection required), **online public 4-player**, and **online private 4-player rooms**.
- Backend: **ASP.NET Core**, prefer supported **.NET 10 LTS** with SignalR, PostgreSQL, one-process Ubuntu/Docker/Nginx deployment first.
- Product: original brand and art; inspired by the publicly described rules of CodeHound's Donkey Master, without copying competitor code, media, UI assets, names, textual descriptions, private protocols or proprietary content.
- Languages: English and Malayalam at initial release.
- Business model: free, optional rewarded ads, later cosmetic purchases; no cash betting, real-money prizes, cash-out, transferable currency or pay-to-win.
- Data boundaries: server-authoritative online matches; local offline engine with strict cross-language test parity.
- Existing input: `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT.md` (source baseline). Do not assume it is approved.

## HARD STOP: phase boundary

**Phase 1 = analyze, document, verify, review, and present options. It does NOT include implementing any functionality.**

Allowed:
- Read repository structure and existing read-only files, inspect any existing code *without changing it*.
- If network lookup is supported and authorized, compare official developer rules/docs; provide exact citations, accessed date, confidence and limitations. If unsupported, state limitation—do not invent observations.
- Create/edit **only Markdown files inside `docs/`** and one Phase 1 summary report. Formatting/linting documentation only is allowed.
- Produce diagrams in Markdown Mermaid, JSON schema **examples inside Markdown**, pseudocode **inside Markdown**, test-case tables, and approval checklist.

Forbidden:
- Do not run `flutter create`, `dotnet new`, `npm create`, migrations, dependency installation, code generators, database seeds, build commands, simulators, emulators, containers, production tools, deploys, billing integrations, or commands that modify game code/config.
- Do not create or modify Dart, C#, SQL, JSON, YAML, assets, package manifests, CI pipelines, API executables or other implementation files.
- Do not silently resolve disputed rules as if official behavior is known.
- Do not mark Phase 1 complete/final/approved without explicit user sign-off on the decision register.
- Do not proceed to Phase 2 or propose to do so within this execution.

## Research foundation and source hierarchy

Primary (publicly available) competitor rules: https://www.donkeymaster.in/ (How to Play, associated with CodeHound branding). Cross-check https://apps.apple.com/app/id1326593908 and https://play.google.com/store/apps/details?id=in.codehound.donkey .

Reliable engineering references: https://docs.flame-engine.org/ , https://learn.microsoft.com/en-us/aspnet/core/signalr/ , https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core , https://developer.android.com/google/play/billing/backend , https://support.google.com/googleplay/android-developer/answer/9857753 and https://developer.apple.com/app-store/user-privacy-and-data-use/ .

Mark each rule as:
- **CONFIRMED** (supported by exact passage/URL),
- **PROPOSED** (our design default),
- **OPEN** (needs test or stakeholder decision).

The primary source confirms 52-card deck, Ace-high order, Ace of Spades *holder* starts, follow suit, off-suit HIT when void, cards discarded without hit, cards awarded to highest-card player with hit, podium finish and last remaining Donkey. It does **not** provide every tie/round boundary/off-suit comparison rule or shuffle algorithm. Treat ambiguity explicitly.

## Required approach

1. Audit existing repository state *read-only*: list relevant files/config, identify whether project is empty or contains existing work. No code edits. Record findings and assumptions.
2. Ingest the attached PRD draft. Create a coverage matrix of product requirements against deliverables and potential unknowns.
3. Compare the official rule text with our proposed rules; fill out a source-evidence table, and highlight contradictions/underspecification before drafting canonical decisions.
4. Draft a versioned and deterministic ruleset with precise legal moves, A♠ opening policy, turn order, hit calculation, trick settlement, finisher settlement, ties, all-players-empty, timeout, disconnect, bot take-over and voids. **When unresolved, present a recommended option plus alternatives—not an invented fact.**
5. Draft explicit state transition diagrams and sequence diagrams: offline round, online round, hit collection, reconnect, simultaneous commands, timeout, worker failover.
6. Produce technology ADRs including .NET 10 LTS; SignalR Dart client compatibility spike; Flame/Flutter rendering and iOS platform constraints; PostgreSQL persistence and Redis optionality; shared fixture parity with Dart/C# engine.
7. Draft proposed entities, indexes, sensitive-data boundaries, versioning and migration plan; no actual migration code.
8. Draft REST/OpenAPI-style and hub contracts **inside Markdown only**: route/method, request/response/event examples, authentication, idempotency, `expectedStateVersion`, `actionId`, sequence, seat filtering, error codes.
9. Produce test-first acceptance matrix: legal/illegal trick fixtures, randomized invariants, cross-language golden tests, load, reconnect, security, offline operation, low-end GPU, Malayalam layout, analytics and platform store requirements.
10. Draft original art and interaction design direction for a 3D-looking tabletop without replicating the competitor's visuals. Include design tokens, card sprite requirements, shadows/perspective, animation budgets, accessibility and low-end fallback.
11. Threat-model cheating, hidden-hand leaks, replay/double-actions, auth abuse, malicious room joins, purchase/virtual-coin fraud, session takeover, backend restart and logging exposure. Document priorities and controls.
12. Present estimates and phases with explicit dependencies, acceptance criteria and risk register. Distinguish planning estimates from tested metrics.
13. Produce review summary and a short *decision worksheet* for owner approval. HARD STOP.

## Documents to create (Markdown only)

```
docs/
  00-product-vision-and-scope.md
  01-prd-and-user-stories.md
  02-verified-rules-and-open-questions.md
  03-canonical-ruleset-v1-draft.md
  04-game-state-machine.md
  05-architecture-and-data-model.md
  06-api-and-realtime-contracts.md
  07-offline-online-parity-strategy.md
  08-security-privacy-and-compliance.md
  09-visual-ux-art-direction.md
  10-qa-test-matrix-and-acceptance.md
  11-development-roadmap.md
  12-risks-and-decisions.md
  ADR/
    0001-flutter-flame-2-5d.md
    0002-aspnet-core-signalr-and-durable-actors.md
    0003-two-language-rules-parity.md
  PHASE_1_REVIEW_REPORT.md
```

Prefer each document to contain Purpose, Scope, Confirmed Information, Proposed Decisions, Open Questions, Risks/Assumptions, Acceptance Criteria, and Cross-References. Diagrams must be valid Mermaid; use explicit states, no complex/imaginary tooling.

## High-priority ambiguous rules that **must** appear in approval worksheet

1. **Deck distribution**: developer says the deck is dealt evenly but also implies non-uniform suit distribution by design. Our proposal: 52-card unbiased CSPRNG shuffle and 13 each. Is any intentional suit balancing/skewing allowed?
2. **Opening card**: must A♠ be played or does its holder merely begin with *any card*?
3. **HIT winner**: highest **leading-suit** card versus highest rank regardless of suit (e.g. lead 8♣, play J♣, 10♣, then off-suit A♥). What happens?
4. **Finisher timing**: exit when hand becomes empty immediately after play, or after round collection/discard resolution? Propose after resolution.
5. **Multiple finishers**: simultaneous ties or deterministic podium order? How is leader chosen if highest-card player exits?
6. **All players empty together**: draw/no Donkey, replay, or another behavior? Propose technical draw/no Donkey.
7. **Last active seat**: how and when declared Donkey, including current trick in progress?
8. **Timeout/disconnect**: grace, auto-play/bot takeover, return of control, fairness and abandonment.
9. **Bot fairness**: can offline or online bot inspect hidden hands? Recommend no; mark bot seats.
10. **Game modes**: four players only at MVP; adding 2/3 players demands explicit dealing/rest-card rule.
11. **Round repetition watchdog**: should pathological matches void? No fabricated winner.
12. **Offline currency**: can offline activity earn online-authoritative rewards? Recommend isolate.

Do **not** hide these in appendices. Put blocking items up front.

## State correctness and security acceptance criteria (write as test cases, not code)

- Exactly 52 unique cards always distributed across hands, current trick, and discard pile.
- No hidden cards in opponent payloads, socket events, HTTP views, logs or telemetry.
- Every player action authenticated, seat-bound, legal, version-checked and idempotent.
- Hit adds round cards to winner hand; no-hit discards round cards; no duplicate settlement.
- Finishers determined after settled trick; terminal result granted at most once.
- Reconnect receives same match sequence, legal own hand, and no opponent hand.
- Duplicate/reordered/out-of-turn events never create double-play or rewards.
- Client manipulations of turn, rank, RNG or virtual balance cannot affect server outcome.
- Offline and online goldens match exactly for the same input state/actions and rule version.
- Game can survive mobile process death and backend restart consistent with documented recovery policy.
- Rewarded ads optional and no ads interrupt active turns; StoreKit/Play Billing handled later with verified backend entitlement.
- Documentation identifies privacy/deletion, iOS ATT and policy compliance gates.

## Final response format after document writing

Return **only** a Phase 1 review summary containing:

1. Read-only repository audit (existing state / constraints).
2. Document manifest (created files, one-line description and link/path each).
3. Reference rules matrix: CONFIRMED / PROPOSED / OPEN and supporting URLs.
4. Architecture decision synopsis with specific tradeoffs.
5. Top ten unresolved decisions with **your recommendation**, alternatives, and risk of guessing.
6. Test strategy and entry/exit criteria.
7. Phase estimates with prerequisites; never imply they are guarantees.
8. Security/quality blockers.
9. Clear status: **`PHASE 1: DOCUMENTS PREPARED — RULES AND SCOPE AWAITING OWNER APPROVAL — NO CODE IMPLEMENTED`** (if documents completed).
10. A final specific approval request covering all blocking rule decisions, then **STOP**.

Never say the rules are an exact reverse-engineered CodeHound implementation unless verified through direct documented/authorized observations. Do not send requests to install packages or generate game code at the end of Phase 1.

---

**End of Phase 1 prompt.**
