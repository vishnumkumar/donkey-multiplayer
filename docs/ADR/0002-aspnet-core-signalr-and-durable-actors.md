# ADR-0002: ASP.NET Core + SignalR with Durable Match Actors

**Status:** PROPOSED  
**Date:** 2026-10-20  
**Decision Makers:** Product Owner / Technical Lead  
**Relates to:** D11 (2.5D), D12 (SignalR client), `05-architecture-and-data-model.md`, `06-api-and-realtime-contracts.md`

## Context

The Donkey multiplayer card game requires:

1. **REST API** for authentication, profile, lobby management, match history
2. **Real-time bidirectional transport** for live match play (card actions, events, reconnection)
3. **Server-authoritative game state** with durable persistence (survives process restart)
4. **Serialized per-match command processing** (no concurrent mutations of the same match)

The project targets a single-developer deployment on a single Ubuntu VPS initially, with a path to horizontal scaling if needed.

## Decision

### Runtime: ASP.NET Core on .NET 10 LTS

Use ASP.NET Core running on **.NET 10**, which is the current Long-Term Support release as of October 2026 (released November 2025, supported through November 2028).

- .NET 10 LTS provides 3 years of security patches and bug fixes
- ASP.NET Core is a mature, high-performance web framework with built-in support for REST APIs, WebSockets, and SignalR
- C# is well-suited for implementing the server-side rules engine with strong typing and pattern matching
- Single `dotnet` toolchain for build, test, publish, and containerization

Reference: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core

### Real-Time Transport: SignalR

Use **ASP.NET Core SignalR** for real-time match communication:

- WebSocket as primary transport; Server-Sent Events and Long Polling as automatic fallbacks
- Built-in connection management, groups (per-match), and reconnection support
- Ordered message delivery within a connection
- Hub method invocation pattern maps naturally to game actions (`PlayCard`, `JoinMatch`, etc.)
- Server-to-client push for events (`CardPlayed`, `RoundResolved`, `MatchCompleted`, etc.)

Reference: https://learn.microsoft.com/en-us/aspnet/core/signalr/

### Match Actor Pattern

Each active match is managed by a **serialized command executor** (match actor):

```
Client → SignalR Hub → Match Actor (serialized queue) → Rules Engine → State Mutation → Persist → Broadcast
```

- One logical actor per match; commands are processed sequentially — no concurrent mutations
- Actor holds in-memory state for low-latency processing during active play
- Every state mutation is durably persisted **before** broadcasting to clients
- Actor is reconstructed from persisted state on process restart or match resume

### Durability: PostgreSQL Event Journal + Snapshots

Match state is durably stored in PostgreSQL:

- **Event journal** (`match_events`): append-only log of every validated action and its outcome, keyed by `(match_id, sequence)`. Includes `actionId` for idempotency.
- **Snapshots** (`match_snapshots`): periodic compressed state snapshots for fast actor reconstruction without replaying entire event history.
- **Match results** (`match_results`): terminal outcomes committed exactly once.

In-memory state is a performance optimization, not the source of truth. On process restart, actors are reconstructed from the latest snapshot + subsequent events.

Redis is **not** used for match durability. It may be introduced later for:
- SignalR backplane (multi-server sticky-session coordination)
- Room registry cache
- Rate limiting counters

But these are optional scale-out concerns, not MVP requirements.

## Dart SignalR Client Compatibility

**This is a known risk requiring a Phase 4 compatibility spike.**

The primary Dart SignalR client packages on pub.dev (e.g., `signalr_netcore`, `signalr_core`) were developed against earlier .NET versions. Compatibility with .NET 10 SignalR hubs, particularly:

- WebSocket transport negotiation
- JWT bearer token authentication flow
- Automatic reconnection behavior
- Binary MessagePack protocol (if used)

...must be verified through contract tests before committing to the package.

### Fallback Plan

If no Dart SignalR package works reliably with .NET 10:

1. **Raw WebSocket + custom JSON protocol:** The client opens a WebSocket to a custom endpoint. The server wraps the same match actor logic but uses manual JSON message dispatch instead of SignalR hub methods. This loses SignalR's automatic reconnection and fallback transports but preserves all game logic.

2. **gRPC bidirectional streaming:** ASP.NET Core supports gRPC; Dart has a mature gRPC client. However, gRPC over HTTP/2 adds complexity for mobile (proxy traversal, battery) and loses SignalR's built-in group management.

The spike should test: connection, authentication, method invocation, reconnection, and message ordering. A passing spike means SignalR is confirmed; a failing spike triggers the WebSocket fallback with 1–2 weeks additional transport work.

## Match Actor Lifecycle

```
MATCH CREATED
  → Actor instantiated (in-memory)
  → Loaded from DB (latest snapshot + events since snapshot)
  → Ready to process commands

DURING PLAY
  → Receive command from Hub
  → Validate (auth, seat, turn, legal move, version)
  → Apply to in-memory state
  → Persist event to PostgreSQL (transactional)
  → Broadcast filtered events to connected clients
  → Advance turn / resolve round

MATCH COMPLETED
  → Commit terminal result (idempotent)
  → Process reward ledger entries
  → Actor may be evicted from memory after cooldown

PROCESS RESTART
  → Active matches reconstructed on demand (first client reconnect triggers load)
  → Snapshot + event replay restores exact state
  → Clients receive StateResyncRequired and replay from last acknowledged sequence
```

### Serialization and Concurrency

- Each match actor processes commands from a **single-threaded queue** (e.g., `Channel<T>` or `SemaphoreSlim(1,1)`)
- No concurrent reads or writes to the same match's mutable state
- If multiple workers exist, match ownership is assigned by consistent hashing or database lease — **no dual writers**
- Fencing: each command checks `expectedStateVersion`; stale commands are rejected

### Worker Failover

For single-process deployment (MVP):
- Process restart reconstructs actors from database
- In-flight commands at crash time are lost; clients retry with same `actionId` (idempotent)
- Match in unrecoverable state → `VOIDED` (no false outcome)

For multi-process (future):
- Database-backed ownership lease with TTL
- Acquiring worker loads state from DB; releasing worker flushes state
- Split-brain prevention via fencing tokens on every write

## Single-Process First, Scale-Out Path

### MVP: Single Process

```
Nginx (TLS) → ASP.NET Core (1 process)
                ├── REST endpoints
                ├── SignalR hub
                ├── Match actors (in-memory + PostgreSQL)
                └── Background services (timeouts, cleanup)
```

- Simple, debuggable, no distributed-systems complexity
- PostgreSQL handles all durability
- Sufficient for early launch scale (est. <500 concurrent users)

### Future: Multi-Process

When scaling beyond one server:

1. **Sticky sessions:** SignalR requires sticky sessions for WebSocket connections. Nginx `ip_hash` or application-level routing.
2. **SignalR backplane:** Redis pub/sub for cross-server event distribution (not for match state durability).
3. **Match affinity:** Route all commands for a match to the same worker. Options: consistent hashing, database lease, or external coordinator.
4. **Stateless REST:** REST endpoints are already stateless; horizontal scaling is straightforward.

Reference: https://learn.microsoft.com/en-us/aspnet/core/signalr/scale

**Key constraint:** Redis pub/sub distributes messages but does not solve authoritative match ownership. Two workers must never process commands for the same match simultaneously.

## Alternatives Considered

| Alternative | Reason Not Chosen |
|---|---|
| **Node.js + Socket.IO** | Viable but loses C# type safety for rules engine; JS/TS rules would need Dart parity testing against a third language. Performance under CPU-bound game logic less favorable. |
| **Go + custom WebSocket** | Excellent performance but no equivalent to SignalR's hub abstraction; would require building connection management, groups, reconnection, fallback transports. Rules engine in Go adds a third language. |
| **gRPC bidirectional streaming** | Mature Dart client, but HTTP/2 streaming is less natural for game event broadcasting; no built-in group/room abstraction; mobile proxy/firewall concerns. |
| **Microsoft Orleans (virtual actors)** | Powerful actor framework but adds significant infrastructure complexity; not justified for single-developer, single-server MVP. Could be adopted later if actor management becomes complex. |
| **Akka.NET** | Similar to Orleans concerns; heavyweight for MVP scale. |
| **Supabase Realtime / Firebase** | Managed services but surrenders server-authoritative game logic; real-time state machine cannot run in client-triggered database triggers with required correctness guarantees. |

## Consequences

### Positive

- Mature, well-documented, high-performance stack
- Single language (C#) for all server logic including rules engine
- Built-in SignalR handles transport complexity (WebSocket, reconnection, groups)
- PostgreSQL provides ACID guarantees for event persistence
- Clear scale-out path when needed
- .NET 10 LTS provides 3 years of stable support

### Negative

- Dart SignalR client compatibility is unverified (mitigated by spike + fallback)
- Single-process means single point of failure (mitigated by durable persistence + restart recovery)
- Actor pattern is custom-built, not framework-managed like Orleans (acceptable for <1000 concurrent matches)
- C# rules engine requires Dart parity testing (see ADR-0003)

### Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| SignalR Dart client fails spike | Medium | WebSocket fallback ready; 1–2 weeks additional work |
| Process crash during active matches | Low | Durable journal; actor reconstruction; void for irrecoverable |
| PostgreSQL becomes bottleneck | Low | Snapshot compaction; connection pooling; match events are small |
| .NET 10 early-lifecycle issues | Low | Pin SDK; follow release notes; standard LTS support |

## References

- .NET support policy: https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core
- SignalR documentation: https://learn.microsoft.com/en-us/aspnet/core/signalr/
- SignalR scaling: https://learn.microsoft.com/en-us/aspnet/core/signalr/scale
- SignalR authentication: https://learn.microsoft.com/en-us/aspnet/core/signalr/authn-and-authz
- `05-architecture-and-data-model.md` — entity design and persistence details
- `06-api-and-realtime-contracts.md` — hub and REST contract specifications
