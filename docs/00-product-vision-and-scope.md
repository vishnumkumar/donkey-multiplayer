# 00 — Product Vision and Scope

**Document ID:** DM-DOC-00  
**Version:** 1.0-RC  
**Date:** 2026-10-21  
**Status:** Phase 1 complete — all rules approved; awaiting final ruleset freeze authorization

---

## Purpose

Define the product vision, target audience, MVP success criteria, business model boundaries, and explicit non-goals for the Donkey multiplayer card game. This document is the north star that all other Phase 1 deliverables derive from.

## Scope

Covers high-level product intent, audience segmentation, measurable MVP goals, post-MVP roadmap boundaries, and hard constraints on monetization and content. Does **not** specify rules (see [03-canonical-ruleset-v1-draft.md](03-canonical-ruleset-v1-draft.md)), architecture (see [05-architecture-and-data-model.md](05-architecture-and-data-model.md)), or detailed feature acceptance (see [01-prd-and-user-stories.md](01-prd-and-user-stories.md)).

---

## 1. Product Vision

**One-liner:** A fast, fair, social four-player Donkey card game with a polished 3D-style appearance, reliable live multiplayer, and fully playable offline mode — built for Android and iOS.

### 1.1 Promise

Deliver a premium-feeling card game that:

- Feels instant: a new user starts an offline match within seconds of first launch, with no account or network required.
- Feels fair: server-authoritative online play, no pay-to-win, no hidden advantage for paying users.
- Feels social: real-time four-player matches with emoji reactions, private rooms with invite codes.
- Feels polished: 2.5D perspective table, smooth card animations, original art direction.

### 1.2 Inspiration and Originality

The game is **inspired by the publicly described rules** of the Donkey card game as documented on the CodeHound Donkey Master website (https://www.donkeymaster.in/). It is an **original, independently branded product** — no competitor code, media, UI assets, names, textual descriptions, private protocols, or proprietary content may be copied.

---

## 2. Target Audience

| Segment | Description | Priority |
|---|---|---|
| Kerala card-game players | Casual/family players familiar with Donkey or similar trick-avoidance games | Primary |
| Broader India casual | Mobile card-game users across India, English-speaking | Secondary |
| Global English | English-speaking casual gamers worldwide | Tertiary |

**Demographics:** Mobile-first (Android-heavy in India), age 13+ (no child-directed content in MVP), affordable smartphones (budget to mid-range), intermittent connectivity in some regions.

**Anti-audience:** Competitive esports players, real-money gamblers, users expecting transferable/redeemable currency.

---

## 3. MVP Goals

| # | Goal | Measurability |
|---|---|---|
| G-01 | New user starts offline with no account or network after install | Binary: achievable from cold install on airplane mode |
| G-02 | Guest user creates/enters online match against other humans; bots fill vacancies only under declared policy | End-to-end four-player cross-device test |
| G-03 | Four-person private room joinable via 6–8 char code; no voice chat in MVP | Room creation → code share → four joins → game completion |
| G-04 | Online and offline rules engines agree on every identical golden fixture | Cross-language Dart/C# fixture suite, 100% pass rate |
| G-05 | No hidden cards or RNG secrets leak to opponents mid-game | Adversarial payload/log/reconnect inspection with zero leaks |
| G-06 | In-game coins are cosmetic/engagement-only — never redeemable, wagered, or transferable | Economy audit: no cash-out, transfer, or wager paths exist |
| G-07 | All match/progression/currency changes safe from retries and duplicate messages | Idempotency test suite: replay identical actionIds, verify no double-grant |

### 3.1 Suggested Post-Beta Performance Targets

These are **aspirational baselines** to be validated through measurement, not launch commitments:

| Metric | Target | Notes |
|---|---|---|
| Crash-free sessions | ≥ 99.5% | Measured via crash reporting SDK |
| Online match completion | ≥ 95% | For healthy connections; excludes deliberate abandonment |
| p95 backend action handling | < 150 ms | Under designed load; load profile TBD |
| Visual frame rate | 60 FPS target | With quality fallback for low-end devices |
| Match resumption | After brief network loss | Grace period and reconnect; exact seconds configurable |

---

## 4. Game Modes (MVP)

| Mode | Players | Network | Authority |
|---|---|---|---|
| **Offline vs Bots** | 1 human + 3 bots | None required | Local Dart rules engine |
| **Online Public** | 4 humans (bots fill if opted-in) | Required | Server C# rules engine |
| **Online Private Room** | 4 humans via invite code | Required | Server C# rules engine |

All modes use the **same versioned ruleset** (`DM-RULES-1.0` once approved). See [07-offline-online-parity-strategy.md](07-offline-online-parity-strategy.md).

---

## 5. Business Model Constraints

### 5.1 Included in MVP Monetization

- **Optional rewarded ads** for noncompetitive cosmetics or fixed bonus coins.
- Frequency caps; no ads during active tricks.
- Interstitials only after completed result screen (initially disabled).

### 5.2 Planned Post-MVP

- Cosmetics inventory and store (verified purchase entitlements).
- Ad-removal purchase option.

### 5.3 Hard Constraints (Non-Negotiable)

| Constraint | Rationale |
|---|---|
| No real-money bets or cash-out | Legal, store compliance, product identity |
| No wagering or real-money prizes | Same |
| No NFT/token exchange | Same |
| No player-to-player currency transfer | Fraud prevention, regulatory |
| No pay-to-win mechanics | Product fairness commitment |
| Offline reward currency isolated from online | Tamper prevention (see [08-security-privacy-and-compliance.md](08-security-privacy-and-compliance.md)) |
| No ads during active tricks | User experience; Google Play policy compliance |
| Purchases via store-compliant billing only | Apple/Google policy requirement |

---

## 6. Explicit Non-Goals (MVP)

The following are **out of scope for MVP** and must not be designed for, depended on, or implied in marketing:

- Friends list and friend invitations
- Leaderboards, ranks, and ranked matchmaking
- Tournaments and seasons
- Cosmetics inventory or store (beyond placeholder)
- Verified purchase entitlement flows (billing integration deferred)
- Voice chat (with or without moderation)
- Spectator mode
- Advanced analytics and A/B testing framework
- 2-player or 3-player variants
- Full 3D physics engine (Unity, Unreal, etc.)
- Administrative modification of cards in live matches
- Compulsory social login
- Private messaging between players
- Landscape game orientation (research spike only; not a dependency)

---

## 7. Localization and Accessibility (MVP)

| Requirement | Detail |
|---|---|
| Languages | English and Malayalam; all UI strings localized |
| Layout | Portrait default; tablet adaptive; landscape deferred |
| Color independence | Suits and turns never conveyed by color alone — glyphs, text labels |
| Motion | Reduced-motion setting respected; animations skippable |
| Touch targets | ≥ 44–48 logical pixels where feasible |
| Screen readers | Suit icon labels, large-number formats accessible |
| Quality presets | Low / Medium / High; avoid GPU-heavy real-time meshes |
| Safe areas | Account for notches, variable aspect ratios, system UI |

---

## 8. Open Questions

| # | Question | Impact |
|---|---|---|
| OQ-00-1 | Exact minimum Android/iOS versions — depends on Flutter SDK version and device analytics | Device support scope, store listing |
| OQ-00-2 | Will the game carry an age rating that requires additional content review (e.g., simulated gambling)? | Store submission, regional legal |
| OQ-00-3 | Is the solo-developer timeline (12–20 weeks indicative) acceptable or does team composition change? | Roadmap and phase gating |

---

## 9. Risks and Assumptions

| Risk / Assumption | Mitigation |
|---|---|
| Audience assumption: Kerala card-game familiarity | Validate via tutorial completion rates and early-access feedback |
| Affordable device performance | Quality presets + low-end GPU fallback; test on budget Android |
| Malayalam localization quality | Native speaker review before release |
| Competitor awareness of similar product | Original branding, differentiated visual identity, no copied assets |
| Regulatory uncertainty around simulated-gambling in India | Legal review before launch; no real-money features |

---

## 10. Acceptance Criteria for This Document

- [ ] Product owner confirms vision statement and audience segments
- [ ] All MVP goals are agreed as testable criteria
- [ ] Non-goals list is exhaustive and agreed
- [ ] Business model constraints are non-negotiable and signed off
- [ ] Open questions assigned owners with target resolution dates

---

## Cross-References

| Document | Relationship |
|---|---|
| [01-prd-and-user-stories.md](01-prd-and-user-stories.md) | Detailed feature requirements and user stories |
| [03-canonical-ruleset-v1-draft.md](03-canonical-ruleset-v1-draft.md) | Game rules this product implements |
| [05-architecture-and-data-model.md](05-architecture-and-data-model.md) | Technical architecture supporting these goals |
| [09-visual-ux-art-direction.md](09-visual-ux-art-direction.md) | Visual identity and premium feel |
| [11-development-roadmap.md](11-development-roadmap.md) | Phased delivery of these goals |
| [12-risks-and-decisions.md](12-risks-and-decisions.md) | Decision register for open items |
| `DONKEY_GAME_PRODUCT_TECHNICAL_SPEC_V1_DRAFT.md` | Source baseline specification |
