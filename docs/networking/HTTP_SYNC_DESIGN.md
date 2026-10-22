# Proposed HTTP scenery synchronization

**Design / not implemented end to end. Updated 2026-10-22.** [Transport facts](TRANSPORT.md) · [Latest evidence](../research/DIAGNOSTICS_050_068.md) · [Recovery](RECOVERY_AND_HISTORY.md) · [Measurements](BENCHMARKS.md)

## Scope and evidence boundary

Initial target: two players collaboratively create, move and delete supported scenery in matching parks. Exchange **accepted object changes**, not the entire park every poll. Existing thousands of unchanged objects require identity/baseline agreement, not continuous retransmission of their transforms. Full guests/staff/economy/terrain/ride simulation is outside this design; matching scenery does not imply deterministic simulation.

The building blocks have passed separately: native GET/text response, sequential A/B messages, HTTP-triggered Create/Move/Delete and native Undo/Redo, guarded redelivery, grouped creation, live-client fault recovery and a controlled saved-checkpoint client reload. Two simultaneous clients, ordinary player-edit capture, remote HTTPS, automatic general durable recovery and resync have NOT passed. This page proposes their integration.

## Roles and data path

```mermaid
sequenceDiagram
    participant A as Host PC2 Lua
    participant R as Owner-controlled relay
    participant B as Guest PC2 Lua
    Note over A,B: Matching baseline and native Kepler context ready
    B->>R: Submit proposed scenery intent
    R->>A: Deliver intent for host validation
    A->>A: Validate, commit once, verify world
    A->>R: Publish accepted ordered event + local outcome
    R->>B: Poll returns ordered accepted events
    B->>B: Map logical IDs, apply once, verify world
    B->>R: Applied-and-verified acknowledgement
    Note over A,B: Retry delivery uses same command ID and immutable payload
```

This is a proposed host-authoritative edit model. The relay coordinates durable delivery and ordering; it cannot decide whether a PC2 preview is actually committable. Define one accepted event sequence at the host and have the relay retain it. A guest proposes intent and waits for acceptance before a durable local effect in the first prototype. A host's already committed originating edit must be recorded as locally applied when its event is echoed, preventing a second local commit.

Ordinary editor input interception is unproven. Start with a scripted sender rather than describing this diagram as capture of normal building actions. If normal UI already committed an edit, it requires a distinct, tested reconciliation/acceptance path; do not automatically replay it on the origin.

## Proposed event envelope

| Field | Purpose |
| --- | --- |
| protocolVersion / operationSchema | Reject incompatible wire interpretation |
| sessionID / sessionEpoch | Isolate parks and invalidate stale runs after reset |
| baselineID / baselineDigest | Ensure peers refer to the same starting world and identity manifest |
| commandID | Immutable unique ID, stable across retransmission |
| acceptedSequence | One authoritative order; local arrival order is not authority |
| originClientID | Track origin and avoid applying an already committed origin effect again |
| objectKey | Mod-owned identity; never transmit a local Part-ID as universal identity |
| operation / resourceKey / worldTransform | Allowlisted CREATE/MOVE/DELETE plus explicit validated data |
| expectedObjectRevision | Reject concurrent/stale edits to the same object |
| canonicalPayloadDigest | Detect conflicting reuse of an ID; define canonical bytes/version first |
| transactionID, if batching | Explicitly distinguish an event group from independent commands |

This schema is a proposal, not a claim that JSON, digest libraries or all transform fields have already been tested in this Lua environment. Existing diagnostics use a small text protocol and a tested rock position. A digest is not authentication.

## Admission, execution and acknowledgement

1. Validate destination session/epoch, baseline, protocol, sender permissions, input bounds and allowlisted resources. Reject non-finite coordinates, unknown operations, excessive body/group sizes and malformed fields. Never execute received Lua/source text.
2. Check commandID against a canonical immutable payload BEFORE effects. Same ID/same content is a duplicate; same ID/different content is a conflict. For a sequence gap, stop at the gap and request the missing event. Existing tests reject out-of-order inputs; a production missing-event buffer/replay service is not yet implemented.
3. Resolve objectKey to the current client's local Part-ID. Look up current Entity-ID when needed; do not persist transient Entity-IDs as identity. Deletion leaves a logical tombstone. CREATE allocates a separate local placement on each peer.
4. Execute one owned editor transaction at a time, with expected revision checked before execution. Keep an in-flight command reservation before native calls, never start a parallel retry while the outcome is unresolved.
5. Wait for entity/preview completion, require CanCommitPreview, reserve the commit attempt BEFORE calling CommitPreview, then wait for the operation and undo-idle state. Verify resource and world transform/mappings over later Advances.
6. Only then acknowledge APPLIED_AND_VERIFIED. HTTP 200 / received / queued is NOT an application acknowledgement. On a completed duplicate return the stored outcome without another editor operation.
7. On uncertainty after commit attempt, pause replication for that operation. Do not guess failure from a disconnected response and blindly commit again.

In-memory deduplication survives the tested network/server failures while the client remains alive (059–062). A controlled completed-checkpoint reload restores the corresponding records in 063. Automatic arbitrary-crash-safe deduplication is not established. It provides **at-most-once commit attempts in the tested lifetime**, not distributed exactly-once effects under crashes.

## Crash and reconnect boundaries

There is no demonstrated atomic transaction joining a PC2 park commit, the relay journal and mod-owned persistent metadata. A crash after native commit but before recording/acknowledging it is an ambiguous outcome. A server remembering a commandID alone cannot resolve whether a restarted client already applied it.

Proposed recovery requires a durable accepted-event journal, client applied watermark, object revision/tombstones, save/baseline identity and an independently verified inventory reconciliation. Persist the logical identity manifest with a defined matching park checkpoint; a manual/server manifest pairing passed in 063, while automatic in-save mod-data integration is still unproven. Reconnect resumes only after epoch/baseline/watermark agreement. On mismatch, stop edits and reconcile or load an agreed checkpoint; do not replay all CREATEs into an unknown park.

Late join should use a known compatible baseline/checkpoint plus subsequent ordered changes, not an unbounded initial event history. Test chunked inventories and repair procedures separately. Saving is a host-controlled explicit checkpoint decision, not a side effect of receiving each event. 063 establishes one controlled local checkpoint workflow, not this distributed metadata/resync workflow.

## HTTP polling, batching and backpressure

Begin with one request in flight per client, bounded event batches and an ordered editor queue. Poll with an applied sequence/cursor; retain undelivered events until the intended consumer acknowledges application. Use timeout/backoff and queue limits, not an infinite tight retry loop. Idle polling frequency and batch budgets must be measured on the actual remote route.

HTTP can carry ordered operations reliably at the transport layer, but reconnects and retries still require application dedup/order/recovery. No request-response protocol makes a game commit atomic. Request delivery latency includes network RTT, poll waiting, native task scheduling, admission and the editor operation.

Diagnostics 048/049 support batching **independent creates**: issue multiple requests with a shared completion token, wait once, edit each result, preview and commit once. Do not merge dependent CREATE/MOVE/DELETE events or unrelated user undo actions into a batch without explicit transaction semantics. One commit for many objects changes the undo boundary and failure handling. 050–054 passed 500-rock groups / 2,701 total rocks; 067/068 profiled real objects and targeted checks. These observations are not a universal frame budget. Large batches and repeated full inventories must remain bounded by measured work and transaction semantics.

Use measurements to choose batch size/frame budget. A localhost 1 MiB response test is not a target packet size and does not establish upload throughput, multi-peer capacity or safe edit rates.

## Internet deployment proposal

Each client connects outward to an owner-controlled HTTPS relay; friends need not share a LAN and ordinarily do not need inbound game-port forwarding in this design. This is untested deployment design. The current runtime evidence is loopback plain HTTP; test native remote HTTPS, certificates, headers, timeouts and reconnect before promising a friend-ready internet mod.

038 used an in-memory diagnostic relay; later 062/063 servers added limited checkpoint journals for their specific recovery tests. None is a production service with complete security/durability guarantees. Do not expose that server unchanged. A public relay needs its own session authentication, authorization, bounded requests, replay/epoch checks, request-rate limits, retention and TLS. `SetGenerateAuthCode(false)` avoids requesting Frontier-generated auth for our endpoint; it does not authenticate our own relay. Production submission should use a defined mutation method/body and avoid sensitive query-string logging. POST/headers/upload support still need an explicit native-API test; successful GET diagnostics do not prove those capabilities.

The relay could be a separate service, or a local sidecar that maintains the remote link while the game uses loopback HTTP. A sidecar is an architectural option, not a tested shipped bridge. It would preserve our validated local HTTP boundary while moving remote security/recovery into original external code.

## Staged acceptance tests

1. Two simultaneous clients: harmless numbered messages both ways, matched session and full task completion.
2. Owner-controlled remote HTTPS route: same test, with explicit latency and timeout evidence.
3. Scripted host CREATE followed by guest application: same logical key, separate local Part-IDs, same resource/world transform, one commit per intended peer.
4. Duplicate/lost response, delayed sequence, conflicting ID and stale revision: no extra game effects; application acks and recovery state recorded.
5. MOVE/DELETE plus independent objects; no origin echo; preview cancellation and local edit ownership.
6. Confirmed ordinary UI edit capture; then persistent mappings, checkpoint/reload, crash/reconnect and late join.
7. Load/soak tests with realistic parks and frame-time measurement, followed by supported resource/undo/transform expansion.

No playable co-op or complete park synchronization is claimed until these gates are met.

## Registry and Undo decisions informed by 064–068

Use the engine's current placement queries with a small logical-key/Part-ID/revision registry. Keep expected-world snapshots used by diagnostics separate from required production state. At 2,701 real rocks the tested query bundles stayed inexpensive; full inventories cost proportionally more and were spread over Advances. 066's 100,000-record test is synthetic, not park capacity evidence.

064 lost native Undo/Redo after the tested reload, although saving alone preserved it. Shared/cross-session history must not depend on identical native stacks. 065's Move counteractions are optional history operations with new IDs, authoritative ordering and expected revision checks; they are not engine-stack reconstruction. Automatic action-data persistence and general Create/Delete counteractions remain open.

066's pruning model rejects old sequences below a checkpoint boundary after removing exact payload records. Production compaction must prove a durable checkpoint and safe retention for all consumers, tombstones and recovery paths before removing evidence. See [recovery/history](RECOVERY_AND_HISTORY.md).
