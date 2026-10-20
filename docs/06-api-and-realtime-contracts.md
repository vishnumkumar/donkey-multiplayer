# 06 — API and Real-Time Contracts

**Document ID:** DM-API-006
**Version:** 0.1 DRAFT — not approved
**Date:** 2026-10-20
**Status:** Phase 1 specification only. No implementation code.

## Purpose

Define the complete REST API and SignalR hub contracts for the Donkey multiplayer backend. Provide JSON request/response examples, authentication flows, error codes, idempotency rules, rate limiting, and seat-filtering requirements. This document serves as the contract between client and server teams.

## Scope

- REST endpoints: auth, profile, lobby/matchmaking, match queries, reporting.
- SignalR hub: client→server methods and server→client events.
- Authentication and authorization flow.
- Error code catalog.
- Rate limiting policy.
- Seat filtering and security rules for all payloads.

---

## 1. Authentication Flow

### 1.1 Guest Registration

Anonymous guest accounts are issued without email or password. The server generates a UUID identity and returns a JWT access token + refresh token pair.

```
POST /api/v1/auth/guest
Content-Type: application/json
```

**Request:**
```json
{
  "displayName": "Player123",
  "preferredLanguage": "en",
  "deviceFingerprint": "android-xxxx-yyyy"
}
```

**Response (201 Created):**
```json
{
  "playerId": "550e8400-e29b-41d4-a716-446655440000",
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g...",
  "expiresIn": 900,
  "tokenType": "Bearer"
}
```

### 1.2 Token Refresh

```
POST /api/v1/auth/refresh
Content-Type: application/json
```

**Request:**
```json
{
  "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2g..."
}
```

**Response (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "bmV3IHJlZnJlc2ggdG9rZW4...",
  "expiresIn": 900,
  "tokenType": "Bearer"
}
```

**Rotation:** Each refresh invalidates the previous refresh token. If an old refresh token is reused (theft detection), all sessions for that player are revoked.

### 1.3 Logout

```
POST /api/v1/auth/logout
Authorization: Bearer <accessToken>
```

**Response (204 No Content)**

Revokes the current refresh token. Access token remains valid until its short expiry.

### 1.4 JWT Structure

```json
{
  "sub": "550e8400-e29b-41d4-a716-446655440000",
  "iat": 1729382400,
  "exp": 1729383300,
  "iss": "donkey-game",
  "aud": "donkey-client",
  "sid": "session-uuid",
  "roles": ["player"]
}
```

- `sub`: player UUID.
- `sid`: session UUID (ties to `player_sessions` row).
- Access tokens: 15-minute lifetime.
- Refresh tokens: 30-day lifetime, single-use with rotation.
- Signed with RS256 (RSA asymmetric) for verification by multiple services.

### 1.5 SignalR Authentication

SignalR connections authenticate via the `access_token` query parameter on WebSocket handshake:

```
wss://api.example.com/hubs/match?access_token=eyJhbGci...
```

The hub validates the JWT on connection. If the token expires during an active connection, the client receives a `TokenExpiringSoon` event and must refresh via REST, then reconnect. The server does NOT drop connections immediately on token expiry if a match is active; a short grace (60s) is allowed.

---

## 2. REST API Endpoints

### 2.1 Profile

#### GET /api/v1/me

Returns the authenticated player's profile.

**Response (200 OK):**
```json
{
  "playerId": "550e8400-e29b-41d4-a716-446655440000",
  "displayName": "Player123",
  "avatarId": "default",
  "preferredLanguage": "en",
  "accountStatus": "active",
  "createdAt": "2026-10-20T10:00:00Z",
  "stats": {
    "matchesPlayed": 42,
    "matchesWon": 15,
    "donkeyCount": 5,
    "currentStreak": 3
  },
  "balance": {
    "coins": 1500
  }
}
```

#### PATCH /api/v1/me

Updates the player's mutable profile fields.

**Request:**
```json
{
  "displayName": "NewName",
  "avatarId": "avatar_02",
  "preferredLanguage": "ml"
}
```

**Response (200 OK):** Updated profile object.

### 2.2 Account Deletion

#### DELETE /api/v1/account

Initiates account deletion workflow. Per Apple/Google requirements, deletion must be available in-app.

**Request:**
```json
{
  "confirmation": "DELETE_MY_ACCOUNT"
}
```

**Response (202 Accepted):**
```json
{
  "message": "Account deletion initiated. Data will be removed within 30 days.",
  "deletionRequestedAt": "2026-10-20T10:00:00Z"
}
```

### 2.3 Rulesets

#### GET /api/v1/rulesets/current

Returns the current active ruleset version and metadata.

**Response (200 OK):**
```json
{
  "rulesetVersion": "DM-RULES-1.0",
  "effectiveDate": "2026-11-01T00:00:00Z",
  "minClientVersion": "1.0.0",
  "summary": "Standard 4-player Donkey with Ace-high ranking"
}
```

### 2.4 Matchmaking Queue

#### POST /api/v1/queue

Join the public matchmaking queue.

**Request:**
```json
{
  "rulesetVersion": "DM-RULES-1.0",
  "clientVersion": "1.0.0"
}
```

**Response (201 Created):**
```json
{
  "queueId": "q-12345678",
  "status": "waiting",
  "position": 3,
  "estimatedWait": null,
  "joinedAt": "2026-10-20T10:00:00Z"
}
```

**Note:** `estimatedWait` is null until we have sufficient data to estimate. We do not invent wait times.

**Errors:**
- `409 ALREADY_IN_QUEUE`: Player is already queued.
- `400 VERSION_MISMATCH`: Client or ruleset version incompatible.

#### DELETE /api/v1/queue/{queueId}

Cancel queue entry.

**Response (204 No Content)**

### 2.5 Private Rooms

#### POST /api/v1/rooms

Create a new private room.

**Request:**
```json
{
  "rulesetVersion": "DM-RULES-1.0",
  "roomSettings": {
    "allowBots": false,
    "disconnectPolicy": "standard"
  }
}
```

**Response (201 Created):**
```json
{
  "roomId": "r-abcd1234",
  "roomCode": "XKWM7P",
  "status": "waiting",
  "rulesetVersion": "DM-RULES-1.0",
  "seats": [
    {"seatNumber": 0, "playerId": "550e8400...", "displayName": "Player123", "isReady": false, "isBot": false}
  ],
  "createdAt": "2026-10-20T10:00:00Z",
  "expiresAt": "2026-10-20T10:30:00Z"
}
```

#### POST /api/v1/rooms/join

Join a private room by code.

**Request:**
```json
{
  "roomCode": "XKWM7P"
}
```

**Response (200 OK):**
```json
{
  "roomId": "r-abcd1234",
  "seatNumber": 1,
  "seats": [
    {"seatNumber": 0, "playerId": "550e8400...", "displayName": "Player123", "isReady": false, "isBot": false},
    {"seatNumber": 1, "playerId": "660f9500...", "displayName": "Player456", "isReady": false, "isBot": false}
  ]
}
```

**Errors:**
- `404 ROOM_NOT_FOUND`: Code invalid or expired.
- `409 ROOM_FULL`: All seats taken.
- `429 TOO_MANY_ATTEMPTS`: Rate limited.

#### POST /api/v1/rooms/{id}/ready

Toggle ready status (private rooms).

**Response (200 OK):**
```json
{
  "seatNumber": 1,
  "isReady": true,
  "allReady": false
}
```

#### POST /api/v1/rooms/{id}/leave

Leave the room. If creator leaves, room may be reassigned or cancelled per policy.

**Response (204 No Content)**

### 2.6 Match Queries

#### GET /api/v1/matches/{id}/snapshot

Returns the current match state for the authenticated player. **Seat-filtered: only own hand returned.**

**Response (200 OK):**
```json
{
  "matchId": "m-98765432",
  "rulesetVersion": "DM-RULES-1.0",
  "state": "WAITING_FOR_PLAY",
  "stateVersion": 15,
  "sequence": 42,
  "currentTurn": 2,
  "leadSuit": "clubs",
  "roundNumber": 5,
  "myHand": [
    {"cardId": "3C", "suit": "clubs", "rank": 3},
    {"cardId": "KH", "suit": "hearts", "rank": 13},
    {"cardId": "7S", "suit": "spades", "rank": 7}
  ],
  "currentTrick": [
    {"seatNumber": 0, "cardId": "8C", "suit": "clubs", "rank": 8},
    {"seatNumber": 1, "cardId": "JC", "suit": "clubs", "rank": 11}
  ],
  "seats": [
    {"seatNumber": 0, "displayName": "Player1", "cardCount": 10, "status": "active", "connectionState": "CONNECTED"},
    {"seatNumber": 1, "displayName": "Player2", "cardCount": 8, "status": "active", "connectionState": "CONNECTED"},
    {"seatNumber": 2, "displayName": "Player3 (you)", "cardCount": 3, "status": "active", "connectionState": "CONNECTED"},
    {"seatNumber": 3, "displayName": "Player4", "cardCount": 0, "status": "finished", "finishPosition": 1, "connectionState": "CONNECTED"}
  ],
  "discardCount": 18,
  "deadline": "2026-10-20T10:05:20Z",
  "serverTime": "2026-10-20T10:05:00Z"
}
```

**CRITICAL:** The `myHand` field contains card details for the requesting player only. Other seats show `cardCount` but NEVER card identities. This filtering must be tested adversarially.

#### GET /api/v1/matches/{id}/history

Returns completed match summary for a participant.

**Response (200 OK):**
```json
{
  "matchId": "m-98765432",
  "rulesetVersion": "DM-RULES-1.0",
  "startedAt": "2026-10-20T09:50:00Z",
  "endedAt": "2026-10-20T10:10:00Z",
  "result": "completed",
  "rounds": 13,
  "results": [
    {"seatNumber": 0, "displayName": "Player1", "finishPosition": 1},
    {"seatNumber": 1, "displayName": "Player2", "finishPosition": 2},
    {"seatNumber": 2, "displayName": "Player3", "finishPosition": 3},
    {"seatNumber": 3, "displayName": "Player4", "resultStatus": "donkey"}
  ]
}
```

### 2.7 Reports

#### POST /api/v1/reports

File a player report.

**Request:**
```json
{
  "subjectPlayerId": "660f9500...",
  "category": "abusive",
  "matchId": "m-98765432",
  "description": "Deliberately timing out every turn."
}
```

**Response (201 Created):**
```json
{
  "reportId": "rpt-111222",
  "status": "pending"
}
```

---

## 3. SignalR Hub Contract

**Hub path:** `/hubs/match`
**Transport:** WebSocket primary; no fallback to long-polling in MVP (mobile clients support WebSocket).
**Authentication:** JWT via `access_token` query parameter.

### 3.1 Client → Server Methods

#### JoinMatch

Called on initial connection or reconnection to subscribe to match events.

```json
{
  "method": "JoinMatch",
  "arguments": [
    {
      "matchId": "m-98765432",
      "lastSeenSeq": 38
    }
  ]
}
```

**Server response:** If `lastSeenSeq` is behind, server sends `StateResyncRequired` followed by a filtered state snapshot and missed events. If up-to-date, server sends `ActionAcknowledged`.

#### PlayCard

Submit a card play action.

```json
{
  "method": "PlayCard",
  "arguments": [
    {
      "matchId": "m-98765432",
      "cardId": "KH",
      "expectedStateVersion": 15,
      "actionId": "a-uuid-12345"
    }
  ]
}
```

**Validation checks (server-side, in order):**
1. Player authenticated and owns a seat in this match.
2. Match state is `WAITING_FOR_PLAY`.
3. It is this player's turn (`currentTurn == player.seat`).
4. `expectedStateVersion` matches server's current version.
5. Turn deadline has not expired.
6. Card exists in player's hand.
7. Card is legal (follows suit if player has suit; any card legal if void in lead suit).

**On success:** `ActionAcknowledged` to sender, `CardPlayed` to all.
**On duplicate actionId:** Returns previously committed `ActionAcknowledged`.
**On failure:** `ActionRejected` with error code.

#### React

Send an emoji reaction (limited to fixed set).

```json
{
  "method": "React",
  "arguments": [
    {
      "matchId": "m-98765432",
      "emojiId": "thumbsup"
    }
  ]
}
```

Rate limited: max 3 reactions per 10 seconds per player.

#### VoteRematch

Vote for a rematch after match completion.

```json
{
  "method": "VoteRematch",
  "arguments": [
    {
      "matchId": "m-98765432"
    }
  ]
}
```

### 3.2 Server → Client Events

Every server event includes a common envelope:

```json
{
  "eventId": "evt-uuid",
  "matchId": "m-98765432",
  "stateVersion": 16,
  "sequence": 43,
  "serverTimestamp": "2026-10-20T10:05:01Z",
  "rulesetVersion": "DM-RULES-1.0"
}
```

#### SeatAssigned

Sent when a player is assigned a seat (lobby → match transition).

```json
{
  "type": "SeatAssigned",
  "seatNumber": 2,
  "matchId": "m-98765432",
  "seats": [
    {"seatNumber": 0, "displayName": "Player1", "isBot": false},
    {"seatNumber": 1, "displayName": "Player2", "isBot": false},
    {"seatNumber": 2, "displayName": "Player3 (you)", "isBot": false},
    {"seatNumber": 3, "displayName": "Player4", "isBot": false}
  ]
}
```

#### MatchStarted

Sent after deal is complete. Contains the player's own hand ONLY.

```json
{
  "type": "MatchStarted",
  "rulesetVersion": "DM-RULES-1.0",
  "firstTurn": 1,
  "myHand": [
    {"cardId": "AS", "suit": "spades", "rank": 14},
    {"cardId": "3C", "suit": "clubs", "rank": 3}
  ],
  "seats": [
    {"seatNumber": 0, "cardCount": 13},
    {"seatNumber": 1, "cardCount": 13},
    {"seatNumber": 2, "cardCount": 13},
    {"seatNumber": 3, "cardCount": 13}
  ],
  "deadline": "2026-10-20T10:00:20Z"
}
```

**SECURITY:** `myHand` contains full card objects only for the recipient. Other seats show `cardCount` ONLY.

#### TurnStarted

Indicates whose turn it is, with deadline.

```json
{
  "type": "TurnStarted",
  "currentTurn": 2,
  "deadline": "2026-10-20T10:05:20Z",
  "leadSuit": "clubs",
  "trickCards": [
    {"seatNumber": 0, "cardId": "8C"}
  ]
}
```

#### CardPlayed

Broadcast to all players when a card is played.

```json
{
  "type": "CardPlayed",
  "seatNumber": 1,
  "cardId": "JC",
  "suit": "clubs",
  "rank": 11,
  "isAutoPlay": false,
  "isBotPlay": false,
  "trickComplete": false
}
```

**Additional private data for the acting player only:**
```json
{
  "type": "CardPlayed",
  "seatNumber": 2,
  "cardId": "KH",
  "myRemainingHand": [
    {"cardId": "3C", "suit": "clubs", "rank": 3},
    {"cardId": "7S", "suit": "spades", "rank": 7}
  ]
}
```

#### RoundResolved

Sent after a complete trick is resolved.

```json
{
  "type": "RoundResolved",
  "roundNumber": 5,
  "hitDetected": true,
  "winnerSeat": 1,
  "leadSuit": "clubs",
  "trickCards": [
    {"seatNumber": 0, "cardId": "8C"},
    {"seatNumber": 1, "cardId": "JC"},
    {"seatNumber": 2, "cardId": "KH"},
    {"seatNumber": 3, "cardId": "2S"}
  ],
  "cardsCollected": 4,
  "seats": [
    {"seatNumber": 0, "cardCount": 9, "status": "active"},
    {"seatNumber": 1, "cardCount": 12, "status": "active"},
    {"seatNumber": 2, "cardCount": 2, "status": "active"},
    {"seatNumber": 3, "cardCount": 0, "status": "finished", "finishPosition": 1}
  ],
  "nextLeader": 1,
  "newFinishers": [
    {"seatNumber": 3, "finishPosition": 1}
  ]
}
```

**Private addition for the round winner (hit only):**
```json
{
  "myUpdatedHand": [
    {"cardId": "JC", "suit": "clubs", "rank": 11},
    {"cardId": "8C", "suit": "clubs", "rank": 8}
  ]
}
```

#### PlayerFinished

Emitted when a player finishes (exits with zero cards after round resolution).

```json
{
  "type": "PlayerFinished",
  "seatNumber": 3,
  "finishPosition": 1,
  "displayName": "Player4"
}
```

#### MatchCompleted

Terminal event. Includes full results. After this, all players' final hands may optionally be revealed.

```json
{
  "type": "MatchCompleted",
  "result": "completed",
  "results": [
    {"seatNumber": 0, "displayName": "Player1", "finishPosition": 1, "status": "finished"},
    {"seatNumber": 1, "displayName": "Player2", "finishPosition": 2, "status": "finished"},
    {"seatNumber": 2, "displayName": "Player3", "finishPosition": 3, "status": "finished"},
    {"seatNumber": 3, "displayName": "Player4", "status": "donkey"}
  ],
  "rewards": [
    {"seatNumber": 0, "coinsDelta": 100},
    {"seatNumber": 1, "coinsDelta": 50},
    {"seatNumber": 2, "coinsDelta": 25},
    {"seatNumber": 3, "coinsDelta": 0}
  ],
  "totalRounds": 13,
  "matchDuration": 600
}
```

#### ConnectionStateChanged

Broadcast when a player's connection state changes.

```json
{
  "type": "ConnectionStateChanged",
  "seatNumber": 3,
  "connectionState": "BOT_CONTROLLED",
  "reason": "disconnect_grace_expired"
}
```

#### StateResyncRequired

Sent to a reconnecting client with authoritative state.

```json
{
  "type": "StateResyncRequired",
  "snapshot": {
    "matchId": "m-98765432",
    "state": "WAITING_FOR_PLAY",
    "stateVersion": 15,
    "sequence": 42,
    "currentTurn": 2,
    "leadSuit": "clubs",
    "myHand": ["3C", "KH", "7S"],
    "currentTrick": [{"seat": 0, "cardId": "8C"}],
    "seats": [
      {"seat": 0, "cardCount": 10, "status": "active"},
      {"seat": 1, "cardCount": 8, "status": "active"},
      {"seat": 2, "cardCount": 3, "status": "active"},
      {"seat": 3, "cardCount": 0, "status": "finished", "finishPosition": 1}
    ],
    "discardCount": 18
  },
  "missedEvents": []
}
```

**SECURITY:** `myHand` contains only the reconnecting player's cards. No opponent hands.

#### ActionRejected

```json
{
  "type": "ActionRejected",
  "actionId": "a-uuid-12345",
  "errorCode": "ILLEGAL_CARD",
  "message": "You must follow the lead suit (clubs).",
  "currentStateVersion": 15
}
```

#### ActionAcknowledged

```json
{
  "type": "ActionAcknowledged",
  "actionId": "a-uuid-12345",
  "newStateVersion": 16,
  "sequence": 43
}
```

---

## 4. Error Code Catalog

### 4.1 REST Error Format

```json
{
  "error": {
    "code": "ROOM_NOT_FOUND",
    "message": "The room code is invalid or has expired.",
    "details": {}
  }
}
```

### 4.2 Error Codes

| Code | HTTP Status | Context | Description |
|---|---|---|---|
| `UNAUTHORIZED` | 401 | All | Missing or invalid JWT. |
| `FORBIDDEN` | 403 | All | Valid JWT but insufficient permissions. |
| `INVALID_REQUEST` | 400 | All | Malformed request body. |
| `VERSION_MISMATCH` | 400 | Queue, Match | Client or ruleset version incompatible. |
| `ALREADY_IN_QUEUE` | 409 | Queue | Player already in matchmaking queue. |
| `ALREADY_IN_MATCH` | 409 | Queue/Room | Player is in an active match. |
| `ROOM_NOT_FOUND` | 404 | Room | Invalid or expired room code. |
| `ROOM_FULL` | 409 | Room | All seats taken. |
| `ROOM_EXPIRED` | 410 | Room | Room TTL exceeded. |
| `TOO_MANY_ATTEMPTS` | 429 | Room | Rate-limited room join attempts. |
| `NOT_YOUR_TURN` | — | Hub | PlayCard sent when it's not this player's turn. |
| `STALE_VERSION` | — | Hub | expectedStateVersion does not match current. |
| `ILLEGAL_CARD` | — | Hub | Card not in hand or does not follow suit. |
| `CARD_NOT_IN_HAND` | — | Hub | Card ID not found in player's hand. |
| `DEADLINE_EXPIRED` | — | Hub | Turn deadline passed before action arrived. |
| `MATCH_NOT_ACTIVE` | — | Hub | Match is not in a playable state. |
| `DUPLICATE_ACTION` | — | Hub | actionId already processed (returns original result). |
| `RATE_LIMITED` | — | Hub | Too many reactions or actions. |
| `INTERNAL_ERROR` | 500 | All | Server error; no game state revealed in response. |

---

## 5. Rate Limiting Policy

| Endpoint / Method | Limit | Window | Scope |
|---|---|---|---|
| `POST /api/v1/auth/guest` | 5 | per minute | per IP |
| `POST /api/v1/auth/refresh` | 10 | per minute | per player |
| `POST /api/v1/rooms/join` | 10 | per minute | per player |
| `POST /api/v1/rooms/join` | 30 | per minute | per IP |
| `POST /api/v1/queue` | 5 | per minute | per player |
| `POST /api/v1/reports` | 3 | per hour | per player |
| Hub `PlayCard` | 20 | per minute | per player (well above normal play rate) |
| Hub `React` | 3 | per 10 seconds | per player |
| Hub `JoinMatch` | 5 | per minute | per player (reconnection bursts) |

Rate limit responses include `Retry-After` header (REST) or `RATE_LIMITED` error code (hub).

---

## 6. Seat Filtering Rules

**Invariant:** No API response, SignalR event, error message, log entry, or crash export ever reveals card identities belonging to another player's hand.

### 6.1 What Each Player Can See

| Data | Own seat | Other seats | After match |
|---|---|---|---|
| Hand card identities | Yes (`myHand`) | NEVER | Optional reveal (post-match) |
| Hand card count | Yes | Yes (`cardCount`) | Yes |
| Played card in trick | Yes (immediately) | Yes (after play broadcast) | Yes |
| Discard pile count | Yes | Yes | Yes |
| Discard pile contents | No | No | Optional |
| Connection state | Yes | Yes (broadcast) | N/A |
| Finish position | Yes | Yes (broadcast) | Yes |
| Match result | Yes | Yes | Yes |
| Shuffle seed | NEVER | NEVER | Optional (replay feature, future) |

### 6.2 Testing Requirements

- Intercept every SignalR event payload; assert no opponent `cardId` or hand details present.
- Intercept every REST response; assert `myHand` only contains requesting player's cards.
- Test reconnect `StateResyncRequired` payload for hand leaks.
- Test error responses: `ILLEGAL_CARD` must not echo the full hand or reveal why (only "must follow lead suit").
- Test log output: no card IDs in application logs.
- Test crash/telemetry exports: no card data.

---

## 7. Idempotency Rules

| Operation | Idempotency mechanism | Behavior on retry |
|---|---|---|
| `POST /api/v1/auth/guest` | `deviceFingerprint` + time window | Returns existing account within 5s window; otherwise creates new. |
| `POST /api/v1/queue` | Player can only be in one queue | Returns existing queue entry. |
| `POST /api/v1/rooms` | Rate limited; no duplicate check | Each call creates a new room (rate limited). |
| `POST /api/v1/rooms/join` | Player+room unique constraint | Returns existing seat if already joined. |
| Hub `PlayCard` | `actionId` unique per match | Returns previously committed acknowledgement. |
| Hub `React` | Rate limited; not idempotent | Each call sends a new reaction (rate limited). |
| Hub `VoteRematch` | Player+match unique constraint | Returns existing vote. |
| Reward grants | `grant_idempotency_key` unique | No duplicate grant. |

---

## 8. Versioning and Breaking Changes

- REST API versioned via URL path: `/api/v1/`, `/api/v2/`.
- SignalR events include `rulesetVersion` in every envelope.
- Breaking event schema changes require a new event type name (e.g., `RoundResolvedV2`) and a deprecation period.
- Client `minClientVersion` is enforced at queue/room join; outdated clients receive `VERSION_MISMATCH` with a prompt to update.
- Backward-compatible additions (new optional fields) do not require version bumps.

---

## Open Questions

| ID | Question | Proposed | Cross-ref |
|---|---|---|---|
| API-OQ-1 | Should the REST snapshot endpoint exist or only use SignalR for state? | Both: REST for initial load and debugging; SignalR for real-time. | 04-game-state-machine |
| API-OQ-2 | Post-match hand reveal: automatic or opt-in? | Opt-in via hub method (future). | 03-canonical-ruleset |
| API-OQ-3 | Match history: how much detail for non-participants? | None; match history is participant-only. | 08-security-privacy |
| API-OQ-4 | WebSocket fallback transport? | No fallback at MVP; require WebSocket. | ADR-0002 |

---

## Acceptance Criteria

- [ ] Every REST endpoint has documented request/response examples and error codes.
- [ ] Every SignalR method and event has documented JSON payloads.
- [ ] No payload example or error response reveals opponent hand data.
- [ ] Idempotency rules documented for every state-changing operation.
- [ ] Rate limits defined for all player-facing endpoints.
- [ ] Authentication flow covers guest creation, refresh rotation, and SignalR connection.
- [ ] Seat filtering rules have explicit test requirements.

## Cross-References

- [04-game-state-machine.md](04-game-state-machine.md) — State transitions that trigger these events
- [05-architecture-and-data-model.md](05-architecture-and-data-model.md) — Entities backing these endpoints
- [07-offline-online-parity-strategy.md](07-offline-online-parity-strategy.md) — Offline engine consumes same event schema
- [08-security-privacy-and-compliance.md](08-security-privacy-and-compliance.md) — Hidden-hand protection enforcement
- [10-qa-test-matrix-and-acceptance.md](10-qa-test-matrix-and-acceptance.md) — Contract test fixtures
