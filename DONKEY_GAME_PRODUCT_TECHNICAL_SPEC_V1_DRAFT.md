# Donkey Multiplayer — Product Requirements & Technical Architecture

**Document ID:** DM-PRD-001  
**Version:** 1.0 (DRAFT — not approved for implementation)  
**Date:** 2026-10-16  
**Target:** Android and iOS, Flutter client, ASP.NET Core backend  
**Visual approach:** Premium 2.5D/3D-style felt table and cards  
**Modes:** Offline vs bots, online public match, online private room  
**Status:** Documentation only. **Do not implement application code before the rule freeze and written approval.**

## 0. Source provenance and certainty

The user-provided competitor is CodeHound's Donkey Master (`in.codehound.donkey`). Its developer-linked game instructions are published at https://www.donkeymaster.in/ and its Apple listing at https://apps.apple.com/app/id1326593908. Current competitor marketing features are on https://play.google.com/store/apps/details?id=in.codehound.donkey.

- **[CONFIRMED]** directly supported by the developer-published rules or product listings.
- **[PROPOSED]** a concrete, testable design decision for *our* original game, not a claim about CodeHound's implementation.
- **[OPEN]** rule or product choice requiring sign-off and possibly hands-on validation in the reference game.

Do not label ambiguous competitor behavior as exact. The standard below is a *complete, implementable proposed ruleset* with unresolved decisions marked, not a reverse-engineered reproduction of CodeHound's unpublished logic. No competitor imagery, branding, text, code or assets may be copied.

## 1. Product vision and success criteria

**Promise:** A fast, fair, social four-player Donkey card game with a polished three-dimensional appearance, reliable live multiplayer, and fully playable offline mode. Designed for Android/iOS with excellent experience on affordable Indian smartphones, initial English and Malayalam localization, and low-pressure monetization.

**Audience:** Indian casual/family card-game players, initial discoverability focus Kerala and broader India, then global English-speaking users. Avoid real-money gambling and redemption.

**MVP goals:**
1. A new user can start offline with no account or network connection after installation.
2. A guest user can create or enter an online match and play against other human players; bots fill optional vacancies only under declared policy.
3. A four-person private room can be joined with a code, with no voice chat in MVP.
4. Online and offline agree on every identical rules-engine golden fixture.
5. No hidden cards or random-seed secrets leak to opponents mid-game.
6. In-game coins are cosmetic/engagement-only, never redeemable, wagered, or transferable.
7. All match, progression and currency changes are safe from retries and duplicate messages.

**Suggested post-beta targets (not guarantees):** crash-free sessions >=99.5%; online match completion >=95% for healthy connections; p95 backend action handling <150 ms under designed load; visual 60 FPS target on mainstream phones with a quality fallback; match resumption after brief network loss. Performance budgets are to be confirmed through baselines rather than treated as promises.

## 2. Product scope and requirements

### 2.1 MVP included

| ID | Capability | Acceptance |
|---|---|---|
| FR-01 | Onboarding | Choose language, guest start, privacy notice, settings. Offline accessible without server. |
| FR-02 | Tutorial | Interactive 4-player example showing follow suit, hit, collected trick, finish order. |
| FR-03 | Offline 4-player | 1 human + 3 bots; Easy/Normal strategy options; save/resume local game. |
| FR-04 | Public online | Four-player lobby, queue, matched table, pre-match loading, real-time turns. |
| FR-05 | Private room | Create 6–8 char invite code, join/leave, host controls, ready check, code expiry. |
| FR-06 | Core rules | Versioned, deterministic state machine and exhaustive validation. |
| FR-07 | Resilience | Sequence/version checks, timeouts, reconnection and match resume, server bot takeover. |
| FR-08 | Visuals | Perspective felt tabletop, raised card shadows, flip/deal/fan/throw/collect animations. |
| FR-09 | UI | Player seat/turn indicators, selectable legal cards, trump-free suit indicator, round history. |
| FR-10 | Accounts | Server-issued guest identity, display name/avatar, optional linked sign-in later. |
| FR-11 | Results | Podium finishing order, last player Donkey, draw/void outcomes, rematch vote. |
| FR-12 | Social-lite | Fixed emoji reactions with rate limits; mute and report player. |
| FR-13 | Observability | Match correlation ID, technical metrics, crash traces without private hands. |
| FR-14 | User controls | Audio/haptics/reduced motion, clear rules, account deletion, safety/reporting. |
| FR-15 | Monetization | Optional rewarded ads only as a controlled beta; no advertisements during active tricks. |

### 2.2 Planned after MVP

Friends and friend invitations, leaderboards/ranks, ranked matchmaking, tournaments and seasons, cosmetics inventory/store, verified purchase entitlements, social voice chat with moderation, stronger bot personalities, spectator mode if privacy design permits, advanced analytics and A/B tests, optional 2/3-player variants after rule confirmation.

**Explicitly out of scope:** real-money bets, cash-out, wagering, NFT/token exchange, player-to-player currency transfer, compulsory social login, private messaging, competitive voice chat in the first release, full 3D physics engine, administrative modification of cards in live matches.

### 2.3 Localization, accessibility, and device support

- English and Malayalam at MVP; all UI strings localized; large-number formats and suit icon labels accessible to screen readers.
- Portrait game layout by default; tablet adaptive; landscape optional research spike (not a dependency for MVP). Account for safe areas, notches and variable display aspect ratios.
- Do not convey suit or turn solely by color. Provide suit glyphs and text, reduced motion, scalable labels and high-contrast cards.
- Low/medium/high visual quality presets; avoid GPU-heavy real-time meshes. Use authored 2.5D assets with depth, lighting, shadows and eased transforms.
- Device minimum Android/iOS versions must be selected based on current Flutter/SDK and analytics support in a formal ADR; do not promise versions until verified.

### 2.4 Screens and user flows

1. Splash and initial asset check.
2. Welcome, language selection, guest play / account continuation.
3. Home lobby: Play Online, Play Offline, Private Room, Profile, Rewards, Settings.
4. Offline setup: ruleset version and bot difficulty; continue saved match.
5. Public matchmaking with cancel, estimated queue status (do not invent time).
6. Private room create, share, join via code; four slots and ready states.
7. Main game table: own hand, 3 opponent seats, lead suit, last play, turn indicator and countdown.
8. Round outcome animation: no-hit discard vs hit collection.
9. Disconnected/reconnecting state and server-driven take-over notice.
10. Match results / ranking / Donkey label / rematch.
11. Tutorial, rule glossary and practice scenario.
12. Profile, achievements placeholder and match history.
13. Settings (language, sound, vibration, quality, accessibility, privacy, delete account).
14. Report/mute UI and network error/support screen.
15. Reward claim UI (conditional on later monetization readiness).

**Flows:**
- Offline: Home → Offline → Tutorial if first run → Deal → Turn loop → Results → Rematch/Exit.
- Online: Guest auth → Queue/Room → Server ready → Server deal → Turn loop → reconnect if needed → Result commit → Results.
- Private: Create room → invite code → seats filled → ready → start when eligibility met; room closed/expired cleanly.

## 3. Core Donkey rules — canonical proposed specification

### 3.1 Source-confirmed rules

**[CONFIRMED]** One normal 52-card deck, no jokers; four suits; ascending numerical 2–10 followed by J, Q, K, A. The entire deck is dealt evenly to four players in standard 4-player games (13 cards each). Holder of Ace of Spades starts the first round. Players follow lead suit if they possess it; otherwise they may play another suit, called a **HIT**. When there is no hit, the largest card of the round leads next and round cards leave hands. When there is a hit, the largest-card player takes all round cards back into their hand and leads next. Players try to exhaust cards; finishers are ranked; last card-holder is the Donkey. The developer site also says deals intentionally avoid equal suit distribution across players, but does **not** publish the algorithm.

Source: https://www.donkeymaster.in/ (How to Play); https://apps.apple.com/app/id1326593908 (description).

### 3.2 Detailed rulings requiring explicit sign-off

| Rule ID | Proposed v1 decision | Certainty |
|---|---|---|
| R-001 | Exactly 4 seated participants at match start. One standard deck; unique cards. | Confirmed 4-player setup / fixed MVP restriction. |
| R-002 | Shuffle uniformly using cryptographic random number generation; deal 13 to each. **Do not implement undisclosed suit-distribution manipulation.** | [PROPOSED]; reference mentions uneven suit distribution, method unknown. |
| R-003 | Holder of A♠ acts first; may lead *any* held card. Leading A♠ is not mandatory. | [OPEN] Source says starts, not must play A♠. |
| R-004 | A round/trick comprises exactly one card from each active participant, in clockwise seat order. Finished participants are skipped on subsequent rounds. | [PROPOSED]. |
| R-005 | First card fixes leading suit. If player holds ≥1 card of leading suit, only those cards are legal; else any card is legal and counts as hit. | [CONFIRMED] core; [PROPOSED] enforcement. |
| R-006 | Rank order A(14) > K(13) > Q(12) > J(11) > 10…2. | [CONFIRMED]. |
| R-007 | For a round with no hit, winner is the highest rank **of leading suit**; played cards go to permanent discard pile. | [PROPOSED] standard interpretation. |
| R-008 | For a round with ≥1 hit, the highest rank **of leading suit** takes **all** played cards; the same player normally leads next. Off-suit hit cards cannot win. | [OPEN] Developer says “largest card” without cross-suit tie policy. Confirm via gameplay. |
| R-009 | A player does not become a finisher immediately upon dropping the last card; resolve the *entire round* first. In a hit, collection may give the player cards again. | [PROPOSED], critical to correctness. |
| R-010 | After each round, any player with zero cards exits and receives next finishing rank in deterministic clockwise seat order relative to the round lead when multiple finish together. | [OPEN] Simultaneous finish behavior unverified; alternative is tied ranks. |
| R-011 | If a round's highest-card player has exited after no-hit resolution, next leader is the first remaining active seat clockwise from that player. | [PROPOSED]. |
| R-012 | If only one active player remains **with at least one card**, finish match and mark that player Donkey; all earlier finishers receive ranks. | [CONFIRMED] objective; [PROPOSED] exact trigger. |
| R-013 | If every participant empties their hand in the same resolution, the match is a **draw/no-Donkey** (no unfair invented loser). | [OPEN]. |
| R-014 | A hit participant with a previously empty hand is not removed until the collection resolves; collected cards fully restore hand size. | [PROPOSED]. |
| R-015 | A player cannot pass, retract, play multiple cards, or act outside their turn. | [PROPOSED]. |
| R-016 | Offline and online use the same ruleset ID and golden test vectors. | [PROPOSED] product invariant. |
| R-017 | A technical infinite-loop watchdog may void/cancel a match, **never award an arbitrary Donkey**. Threshold/compensation TBD. | [OPEN]. |

**Do not start logic implementation until R-003, R-008, R-010, R-013 and the dealing-distribution policy R-002 are approved.** A short controlled competitor gameplay study could resolve ambiguities, but engineering must not misrepresent guesses as CodeHound's rules.

### 3.3 Reference round examples / tests

- **No-hit:** Leader 8♣, other active players 10♣, Q♣, 2♣. No hit → Q♣ wins lead; all four cards to discard pile.
- **Hit:** Leader 8♣, J♣, 10♣, 2♥ (last player has no clubs). HIT → under proposed R-008, J♣ player takes four cards and leads next.
- **Follow-suit enforcement:** Leader 8♣; player holds 3♣ and A♥ → A♥ is illegal. Reject with no turn advance.
- **Off-suit higher-rank hit:** Leader 8♣, J♣, 10♣, A♥ → proposed winner J♣; this is a priority **validation fixture** because published rules leave it ambiguous.
- **Round-end empty hand with hit:** Player plays final 2♥ off-suit. Collection goes to leading-suit winner; evaluate all empty players **after** collection; if the 2♥ player remains empty, mark finisher.
- **No-hit winner runs out:** Round winner has zero cards. Winner exits; first remaining clockwise participant leads next.
- **Two active seats:** Trick collects two plays, skipping earlier finishers.
- **All hands emptied:** Return DRAW/no-Donkey; do not assign arbitrary final loser (R-013 pending approval).
- **Duplicate command:** Same action ID produces same acknowledged result; no double card play.

### 3.4 State invariants

All states, including transient pending round states, must satisfy:

1. Exactly 52 distinct card IDs are partitioned among players' hands, cards played in the open round, and permanent discard; no missing/duplicate cards.
2. For hit resolution, all cards in the round move to one winner hand; no discards are made for that round.
3. For no-hit resolution, all cards in the round move to permanent discard.
4. Every current turn belongs to a seat active in the current round, never to a finished seat.
5. No cards become visible in other players' hands until match completion, except cards legally played or tutorial-only content.
6. Exactly one event sequence advances the match at a time; action versions are monotonic.
7. Finish order is stable; no player re-enters after confirmed exit.
8. Terminal results are immutable, committed at most once per match.
9. All validations precede mutations; rejected actions cannot change hand/count/deadlines.
10. External rewards are never calculated from client-submitted winner/rank.

## 4. Game and lobby finite-state machines

### 4.1 Match lifecycle

`CREATED → JOINING → READY → DEALING → WAITING_FOR_PLAY → RESOLVING_ROUND → WAITING_FOR_PLAY → … → COMPLETED`

Additional terminal or exceptional states: `CANCELLED`, `EXPIRED`, `VOIDED`, `SUSPENDED_FOR_RECONCILIATION`.

- `CREATED`: lobby allocated, ruleset ID fixed, capacity=4.
- `JOINING`: participants reserve seats; validate identity, rate limits and room admission.
- `READY`: room full, public auto-ready or private ready check; lock seats.
- `DEALING`: server generates shuffle, deals, creates initial versioned snapshot, identifies A♠ holder.
- `WAITING_FOR_PLAY`: only current seat may issue `PlayCard` before deadline.
- `RESOLVING_ROUND`: deterministic resolution after all active round seats played; no unrelated card actions accepted.
- `COMPLETED`: results committed once, ledger processed idempotently, players may rematch.
- `CANCELLED`: e.g. empty private room before start.
- `EXPIRED`: unstarted room TTL exceeded.
- `VOIDED`: irrecoverable server/match integrity fault; no competitive outcome or stake loss.
- `SUSPENDED_FOR_RECONCILIATION`: recoverable persistent-state/worker problem; read-only until replay/resume.

**Orthogonal connection states per seat:** `CONNECTED`, `DISCONNECTED_GRACE`, `BOT_CONTROLLED`, `RECLAIM_PENDING`; connection changes do not mutate hand ownership or game phase.

### 4.2 Turn resolution algorithm (behavioral pseudocode, NOT production code)

1. Receive `PlayCard(matchId, cardId, expectedStateVersion, actionId)`.
2. Authenticate player, seat, match membership, ruleset, current state, turn, deadline, legal card.
3. Reject stale/duplicate/invalid actions without state change; for duplicate `actionId`, return previously committed acknowledgement.
4. Atomically remove one card from actor hand and append to current trick; advance match version.
5. If trick incomplete, advance turn to next active seat; broadcast only public event plus private hand for actor.
6. If trick complete, determine hit flag and round winner according to approved `R-008`.
7. Move trick to winner hand (hit) OR discard (no hit).
8. Determine new finishers only *after* movement; assign approved finishing semantics.
9. If terminal condition met, commit final result once; else determine next leader, initialize next trick and deadline.
10. Commit event to durable match log (or durable snapshot/event journal as approved), publish sequenced, seat-filtered views.

### 4.3 Connection and inactivity defaults

Proposed configurable initial values, **not verified competitor behavior**: 20-second action deadline; 15-second disconnection grace; after a deadline, server executes a legal bot move or a specified auto-play policy, with an audit event; reconnecting player receives an authoritative snapshot and may reclaim control at a safe round boundary. If bot takeover is enabled for public matches, it must be disclosed; never simulate a bot as an undisclosed real person. Private host may choose stricter room policies before game starts. Device clocks never adjudicate timeouts.

**Recovery:** match state must survive backend process restart, using an append-only event journal + compact snapshots in database (small-volume MVP), or an equivalently reliable transactional design; a lost in-memory dictionary alone is unacceptable for ranked/live matches. One actor/serialized command executor per match; no concurrent mutations of same match. Durable event + result commit before broadcasting to prevent ghost plays. On reconnect, return an authorized filtered snapshot plus events after acknowledged sequence. Network retry always uses original `actionId`.

## 5. Architecture

### 5.1 Recommended stack

| Layer | Choice | Rationale |
|---|---|---|
| Android/iOS client | Flutter stable, Dart | Single maintainable codebase; efficient iteration. |
| Card/table rendering | Flame + Flutter overlays + bespoke 2.5D assets | App UI in Flutter, fluid GPU-backed transforms, no heavyweight 3D engine needed. |
| Offline engine | Pure Dart deterministic rule engine | No network requirement; independently testable. |
| Online service | ASP.NET Core **.NET 10 LTS** | Current LTS as of Oct 2026, maintainable server. |
| Real-time | ASP.NET Core SignalR | Ordered server-issued actions/events over WebSockets with fallback policy. |
| REST | ASP.NET Core minimal endpoints/controllers | Auth, profile, lobby discovery, history, administrative services. |
| Persistence | PostgreSQL + EF Core | Users, rooms, matches, snapshots/events, reports, progression, entitlements. |
| Cache | Optional Redis after load measurement | Room registry and scale-out only if necessary; Redis is not sole match durability. |
| Edge/deploy | Ubuntu, Docker Compose, Nginx HTTPS/WSS | Low operational overhead and transparent self-hosting. |
| Push notifications | FCM/APNs later | Match invites, reminders (not core gameplay transport). |
| Admin web | React + TypeScript later | Internal moderation/configuration/reporting. |
| Analytics | Privacy-conscious app/server event stream | Dropoff, crashes, stability, retention and conversion. |

Microsoft .NET support: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core . Flame docs: https://docs.flame-engine.org/ . SignalR scaling: https://learn.microsoft.com/en-us/aspnet/core/signalr/scale .

**Flutter SignalR SDK caveat:** Dart packages such as `signalr_netcore` need a compatibility spike against .NET 10, WebSockets, mobile resume and JWT refresh before selection. The package listing says it was tested on older .NET versions; do not assume current compatibility without contract tests. Alternative transport/adaptor choices remain open.

### 5.2 Logical topology

```
Android Flutter/Flame        iOS Flutter/Flame
        | HTTPS / WSS (authenticated, versioned)
        +-------------+
                      |
                 Nginx / TLS
                      |
               ASP.NET Core host
           REST API + SignalR hubs
                      |
       Auth, Rooms, Matchmaking, Actor
               Rules Engine (C#)
                      |
              PostgreSQL (durable)
    identities | match log | snapshots | results
                      |
            Redis (optional future)

Offline: Flutter app → Dart rules engine → local-only state
         Shared ruleset spec + cross-language JSON fixtures
```

**Separation:** Flutter/Flame is animation/rendering, not online authority. Hub methods accept *player intentions*; state machine and randomness reside server-side. Rules are implemented in two languages for offline availability; prevent divergence using exhaustive shared fixture suite generated from the signed ruleset, cross-language parity CI, and a ruleset-version field on every match. Offline rewards/progression are clearly segregated from online authoritative balances.

### 5.3 High-level service modules

- Identity: anonymous guest issuance, secure refresh rotation, linked accounts later, deletion, device reassociation.
- Lobby/matchmaking: queue TTL, duplicate-queue exclusion, 4 seats, room code entropy/expiry, private ready check.
- Match coordinator: per-match serialized actions, monotonic seq/versions, timeout scheduler, reconnection.
- Rules: pure card/turn/trick/finish engine, no database or networking dependencies.
- Bot engine: legal actions via same validator, difficulty policies; deterministic seed in simulations.
- Persistence: write-ahead event journal, snapshots, replay/compaction, match history.
- Economy: append-only ledger and idempotent reward grants; not critical path for card actions.
- Moderation: mute/report, user block, abuse logs with retention limits.
- Telemetry: pseudonymized structured event logs, error budgets and dashboards.

### 5.4 Core storage entities (logical, no migrations yet)

- `players`: UUID, account status, public name/avatar, preferred language, created/last seen, deletion workflow.
- `player_sessions`: refresh token hash, device binding metadata, expiry, revoked marker.
- `lobbies`: UUID, mode, room-code hash, privacy, creator, ruleset, status, expiration.
- `lobby_seats`: room/seat, player UUID or bot marker, ready, membership.
- `matches`: UUID, room, ruleset version, state, sequence, start/end, result hash.
- `match_events`: match UUID + sequence PK, type, actor seat, payload, actionId unique, time, integrity hash. **Sensitive server-only.**
- `match_snapshots`: match UUID + sequence PK, encrypted/restricted full state, schema version, checksum.
- `match_results`: match UUID + player unique key, position/status, finish sequence, void reason.
- `reward_ledger`: append-only unique ledger transaction ID, player, source match, delta, reason, grant idempotency key.
- `player_reports`: report IDs, subject, report category, created time, moderation outcome.
- `asset_catalog`: cosmetics and pricing metadata only after monetization scope approved.

Indexes: normalized room code uniqueness for active rooms, `matches(status, started_at)`, event order `(match_id, sequence)`, ledgers by player/time and unique reward-grant key. Sensitive state stored with least access; request-filtered DTOs prevent leak.

### 5.5 API and hub contract drafts (no code)

REST design:
- `POST /api/v1/auth/guest`, `POST /api/v1/auth/refresh`, `POST /api/v1/auth/logout`, `DELETE /api/v1/account`
- `GET/PATCH /api/v1/me`, `GET /api/v1/rulesets/current`
- `POST /api/v1/queue`, `DELETE /api/v1/queue/{queueId}`
- `POST /api/v1/rooms`, `POST /api/v1/rooms/join`, `POST /api/v1/rooms/{id}/ready`, `POST /api/v1/rooms/{id}/leave`
- `GET /api/v1/matches/{id}/snapshot`, `GET /api/v1/matches/{id}/history` (participant-filtered)
- `POST /api/v1/reports`

Hub at `/hubs/match`:
- Client→server: `JoinMatch(matchId,lastSeenSeq)`, `PlayCard(matchId,cardId,expectedVersion,actionId)`, `React(emojiId)`, `VoteRematch`.
- Server→client: `SeatAssigned`, `MatchStarted`, `TurnStarted`, `CardPlayed`, `RoundResolved`, `PlayerFinished`, `MatchCompleted`, `ConnectionStateChanged`, `StateResyncRequired`, `ActionRejected`, `ActionAcknowledged`.
- Events carry `eventId`, `matchId`, `stateVersion`, `sequence`, `serverTimestamp`, `rulesetVersion`, and visibility class. Never trust a client-submitted seat, balance, outcome or clock.
- Match events and snapshots must be per-recipient filtered; adversarial test that opponents cannot inspect one another's hands in payloads, logs, reconnect responses, or errors.
- Maintain an OpenAPI contract for REST and language-neutral JSON event schema with versioned breaking changes.

### 5.6 Multiplayer correctness and scalability

- Single match-authority owner with per-match serialized command queue/actor. Ensure ownership lease/fencing if operating multiple workers; no dual writers during failover.
- PostgreSQL transactions for durable event and state updates; outbox pattern or equivalent for post-commit broadcast. Prioritize correctness before throughput.
- For single VPS use one backend process and Postgres first. When adding nodes, decide on sticky sessions and transport according to official SignalR scale guidance; Redis pub/sub alone does not solve authoritative match ownership.
- Distinguish network latency from visual animations: only event-driven game actions go to server; 60 FPS rendering never sends per-frame positions.
- Rate limit lobby creation, invites, join attempts, auth, hub methods; support queues separated by ruleset/build compatibility.
- Introduce gradual load tests: 25, 100, 500, 1,000 concurrent clients (synthetic targets, not asserted capacity), record p50/p95 latency, CPU/RAM, connection resets and database load.

## 6. Bot architecture

- Bots never bypass card rules: they query `GetLegalMoves` then select one; move is submitted through the same transition executor.
- Easy bot: random legal moves with seeded RNG (not used for secure shuffling).
- Normal bot: rank conservation, suit exhaustion and estimated hit probability, only information known to that player (no hidden opponent hands).
- Advanced bot deferred. Anti-cheat requirement: even an offline bot may know full state mechanically, but its decision policy should be restricted to information visible to the simulated seat for fairness.
- Time-bounded decisions (suggest 400–1,200 ms animated delay) are cosmetic and never extend online server deadlines without authorization.
- Replay bot decisions deterministically using ruleset version and bot policy seed.

## 7. Visual and motion design system

**Direction:** Original premium card-club 2.5D, dark emerald/navy felt, muted gold/copper accents, softly lit card edges, subtle highlights, realistic shadows; avoid copying reference game's brand or screenshots.

- Table uses a pre-rendered perspective surface (not physics-heavy geometry); adaptive four seat anchors.
- Cards are original licensed/rendered PNG/WebP texture atlases; 3D *appearance* via layered shadows, transform matrices, highlights and card-back flips.
- Motion: deal fan 150–250 ms per card burst; play 180–280 ms; round collect 450–650 ms; respect reduced motion; all durations adjustable after usability testing.
- Input: tap to select, tap again/Play button to confirm (accessibility), optional drag with cancel; illegal cards dimmed with explanatory hint; ≥44–48 logical-pixel touch targets where feasible.
- UI overlays: own hand bottom, other seats top/left/right, lead suit and countdown highly legible; no persistent banner ad in active table.
- Audio/haptics: shuffling, play, hit, podium, mute toggle; throttle repeated haptics; no forced sound.
- Art deliverables: table background portrait and wide tablet crops, 52 card faces + back variants, avatars, 4 suit tokens, card shadow layers, animation reference, UI component atlas, sound pack with rights.
- Quality: fallback to static table and reduced transparency on weaker GPUs; no unbounded texture memory growth.

## 8. Security, privacy and store compliance

### 8.1 Threat model and defenses

| Threat | Defense / acceptance |
|---|---|
| Client fabricates card, rank or outcome | Server checks current hand, legal moves and active state; result never client authoritative. |
| Replay/duplicate or reorder actions | UUID `actionId`, expected version, unique constraint and idempotent ack, monotonic sequence. |
| Hidden-hand exposure | Seat-filtered DTOs; adversarial socket, REST, logs and crash-export tests. |
| RNG manipulation | OS CSPRNG and unbiased shuffle; secret seed server-only before completion; independently test distribution. |
| Multi-device double action | Session/seat ownership, per-match serialization, atomic version checks. |
| Fake reward or purchase | Signed ad callbacks/validated reward grants; verified receipt server-side; append-only ledger. |
| Bot identity deception | Mark bot-controlled seats clearly; match results identify takeover policy. |
| Brute-force private room codes | High entropy, short TTL, membership limits, IP+account rate limits and attempts caps. |
| Malicious chat/reactions | Fixed emoji set, blocking/reporting, per-user limits; voice/social deferred. |
| Credential leakage | TLS/WSS, short-lived tokens, rotation, hashed refresh tokens and OS secure storage. |
| Backend failover | Durable journal, snapshot checksums, fencing/no dual writers, safe void if impossible to reconcile. |

### 8.2 Policy and data handling

- Privacy policy, terms, deletion workflow (including guest accounts), data retention matrix, consent for ads/analytics, privacy labels and Data Safety disclosures must be complete pre-launch.
- If tracking occurs on Apple platforms, comply with ATT; prefer contextual/nonpersonalized ads where consent absent. See https://developer.apple.com/app-store/user-privacy-and-data-use/ .
- Apple requires account deletion initiation inside apps supporting account creation, including auto-created guest accounts. See https://developer.apple.com/support/offering-account-deletion-in-your-app/ .
- Google Play's ads policy prohibits unexpected full-screen ads disrupting active gameplay; use only obvious opt-in rewarded placements and post-results interstitial experiments. See https://support.google.com/googleplay/android-developer/answer/9857753 .
- Purchases on Android/iOS must use store-compliant billing paths and backend validation/entitlement controls. See https://developer.android.com/google/play/billing/backend .
- Do not permit money wagers, cash-out, transferable currency or real prizes. Review simulated-gambling ratings, child-directed app restrictions, applicable Indian rules and regional legal constraints before launch; legal/compliance review is separate from this engineering document.
- Encryption at rest for sensitive match state and token records; secure offsite backups, audit access, least privilege, restore drills.

## 9. Analytics, monetization and operational design

**Beta analytics:** install/open (privacy-permitted), tutorial completion, offline match start/end, queue time, room join failure, abandonment, play validation failures, reconnects, bot takeovers, result transitions, crash/performance buckets, optional reward impression/earned. Use coarse/pseudonymous identifiers and consent appropriate to geography.

**Revenue v1:** optional rewarded ads for noncompetitive cosmetics or fixed bonus coins, frequency caps, no tricks interrupted. Keep offline reward currency distinct from server currency to prevent offline tampering. Interstitials only after completed result screen, with strict frequency cap and opt-out purchase later; initially can remain disabled. Ads never required to complete core match.

**Economy:** no wagering, no transfers and no conversion to money; grants use `(sourceType,sourceId,playerId)` unique idempotency key. Ads and purchases require server confirmation, refund reconciliation and transparent pending states. Do not show a purchase UI until verified billing and support flows are ready.

**Operational:** daily database backup (and periodic restore test), 7–30-day short-lived technical log policy subject to privacy/legal retention analysis, uptime alerting, deployment rollback, minimum CI checks, segregated development/staging/production secrets, healthcheck and graceful in-progress match recovery.

## 10. Test strategy, matrices and release gates

### 10.1 Rules engine tests

- Table-driven tests for all card ranks/suits and legal follow-suit behavior.
- Golden fixtures for: no-hit, one/multiple hits, off-suit Ace hit, a hit on the final card, simultaneous zero hands, last remaining seat, clockwise skip, finished-leader replacement, timeout autoplayer, and match terminality.
- Property-based test every valid state transition: 52-card conservation, unique ownership, monotonic sequence, no invalid turn, no unauthorized reveal.
- Simulate at least 100,000 deterministic matches across random deals, random legal bots, disconnects/timeouts; assert no invariants, infinite loops or unbounded snapshots. Any watchdog hit must produce diagnostic seed and an explicit test case.
- Verify Dart offline and C# online return identical canonical state hashes/events for each shared JSON fixture at the same ruleset version; beware platform RNG differences by using fixed input decks instead of comparing two RNG algorithms.

### 10.2 Client and integration tests

- Flutter widget/golden UI across small phones, tablets, notches, text scales, Malayali labels, dark mode and reduced motion.
- REST/hub contracts: authenticated joins, malformed payloads, duplicate `actionId`, stale versions, replay, out-of-order and unauthorized seat.
- Network: 2G/3G latency profiles, packet loss, Wi-Fi/mobile switching, airplane mode, background/foreground, forced app termination, rejoin after 10s/60s and server restart.
- Cross-platform: real Android low/mid devices and at least one current iPhone; ensure graphics drivers don't corrupt card display.
- Load: scripted 25→100→500→1000 clients (measure only; do not promise these values) plus soak tests, slow clients and giant room-code attempts.
- Security: fuzz card inputs and API, inspect all event payloads for secrets, ensure HTTP 401/403 behavior, verify rate limits, purchase and reward idempotency.
- Operations: backup restore, schema backward compatibility, versioned client join refusal, disaster recovery and error rollback.

### 10.3 MVP entry/exit criteria

**Entry:** approved ruleset `DM-RULES-1.0`, decisions signed, approved API contract, documented schema, approved visual direction, stable CI targets.

**Exit:** P0/P1 bugs resolved; rules suite 100% passing; 100,000 simulated matches without conservation failures; zero unauthorized hidden-card leaks; end-to-end four-player cross-device completed matches; reconnection/restore demonstrated; store disclosures and deletion flows reviewed; cost/performance report available; launch rollback tested.

## 11. Milestone and delivery roadmap

| Phase | Name | Deliverable | Gate |
|---|---|---|---|
| **1** | Product & rule freeze | PRD, verified vs open rules, state machine, architecture ADRs, contracts, tests, design brief. **Docs only.** | Written approval of ambiguities; no code. |
| **2** | Deterministic engines | C# authoritative domain, Dart offline domain, common golden fixtures, simulation harness. | Rules parity and invariants green. |
| **3** | Offline playable prototype | Flutter 2.5D table and bot game with full offline tutorial and local save. | Multiple full offline matches on both platforms. |
| **4** | Backend foundation | Guest auth, rooms, durable match journal, API, match actor and SignalR transport compatibility spike. | Security/integration/recovery suites green. |
| **5** | Online multiplayer | Queue, private room, four cross-device players, reconnect/bot takeover, results. | Load/network fault testing green. |
| **6** | Polish and safety | Visual/sound optimization, localization, moderation, settings, accessibility, telemetry. | Mobile QA and accessibility report accepted. |
| **7** | Monetization and operations | Optional rewarded ads, verified rewards, purchase framework if approved, operations and admin essentials. | Store and financial integrity checks green. |
| **8** | Closed beta and release | Staged Android/iOS releases, crash analytics, security audit, support and rollback. | Release checklist and performance baselines met. |

**Indicative solo-developer planning range:** 12–20 weeks for a tightly scoped tested MVP, depending on asset quality, concurrency requirements, and test devices; excludes advanced social/tournaments. Calibrate this after Phase 1 estimates, not as a fixed commitment.

## 12. Phase 1 concrete deliverables and approval gate

Phase 1 is **specification only**. Produce these repository documents; do not create project scaffolding, code, tests, migrations, assets, Docker resources or dependencies:

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

Report **PASS (docs complete; awaiting decision)** vs **BLOCKED** separately; never claim “spec finalized” merely because documents were written. Required review meeting/sign-off must expressly settle R-002, R-003, R-008, R-010, R-013 and bot/disconnect policy. Once user approves, issue `DM-RULES-1.0` immutable ruleset and only then authorize Phase 2 implementation.

## 13. Decision register — high-impact unresolved items

| ID | Question | Proposed default | Evidence/action |
|---|---|---|---|
| D01 | Shuffle distribution? | Uniform 52-card shuffle; no forced suit imbalance. | Reference says uneven suit distribution deliberately; algorithm not available. Need decide originality vs fidelity. |
| D02 | Must A♠ be opening card? | Holder starts with any card. | Ref says holder begins; observe opponent game and approve. |
| D03 | In a hit, can off-suit Ace beat led-suit Jack? | No; highest led suit takes trick. | Developer uses “largest card” without cross-suit resolution. Verify with test play. |
| D04 | Multiple players finish in same round? | Rank deterministically clockwise; could instead award ties. | No official edge rule. |
| D05 | Everyone runs out together? | Technical draw/no Donkey. | No official edge rule. |
| D06 | Human timeout and disconnect? | 20s deadline + 15s grace + disclosed bot control. | User experience decision. |
| D07 | Competitive offline progression? | Isolated local rewards; no upload into online economy. | Security requirement. |
| D08 | Room start with unfilled seats? | Private rooms require 4; host may add visible bots. | Product decision. |
| D09 | Online bot vs real players? | Bots disclosed; only queue fallback after user opts in. | Fairness/transparency. |
| D10 | Real 3D vs 2.5D? | Flame-rendered 2.5D appearance, no Unity. | Performance/cost preference. |
| D11 | Client real-time package? | Compatibility-spike current pub.dev SignalR client. | May not fully support .NET 10. |
| D12 | Regional compliance/reviews? | India launch with non-gambling economy; later other countries. | Requires store/legal verification. |

### Official/reference links

- CodeHound Donkey Master rules: https://www.donkeymaster.in/
- CodeHound Apple listing: https://apps.apple.com/app/id1326593908
- Play listing: https://play.google.com/store/apps/details?id=in.codehound.donkey
- Flame docs: https://docs.flame-engine.org/
- .NET version support: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core
- SignalR scaling: https://learn.microsoft.com/en-us/aspnet/core/signalr/scale
- SignalR auth: https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz
- Google Play ads: https://support.google.com/googleplay/android-developer/answer/9857753
- Play Billing backend: https://developer.android.com/google/play/billing/backend
- Apple tracking: https://developer.apple.com/app-store/user-privacy-and-data-use/
- Apple account deletion: https://developer.apple.com/support/offering-account-deletion-in-your-app/
