# Open questions and next experiments

**Updated 2026-10-22 through diagnostic 049; 050 pending review.** [Status](STATUS.md) · [Transport](../networking/TRANSPORT.md) · [Proposed protocol](../networking/HTTP_SYNC_DESIGN.md)

## Immediate acceptance gates

1. **Simultaneous clients:** two independently authorized clients exchanging harmless numbered messages at the same time. Sequential A/B/A-again exchange passed; it is not this test.
2. **Remote route:** native HTTP over an owner-controlled internet HTTPS relay, certificates/headers/timeouts and friend-friendly setup. A package was prepared, not tested with a remote friend.
3. **Cross-client replay:** matching baselines and logical object keys; one scripted event creates separate local parts with matching resource/transform and one intended commit per peer. Test origin echo avoidance.
4. **Edit capture:** observe an ordinary confirmed player action without shipping previews or double-applying the origin's already committed change.
5. **Identity and recovery:** persist per-client logical-key mappings/revisions/tombstones and park checkpoint IDs; handle commit-before-ack crashes, gaps, reconnect and late join without blind CREATE replay.

## Transport and performance coverage

- POST/body/header support, remote TLS behavior, redirects, compression, connection reuse and long polling remain untested. Loopback GET/text is established.
- Request cancellation, failures during world/owner transitions, bounded queues/backpressure and soak tests.
- Offline fresh uncached profile, Steam outage and permanent Frontier service shutdown: not established by success with a ready Kepler context and Frontier disconnected.
- Review 050 before any claim about 100/250/500-object groups or 2,701-object parks. Measure synchronous frame stalls with a better timing basis; zero game-clock phase time is not zero cost.
- Preserve batch transaction/undo semantics; do not coalesce dependent events merely for speed.
- Different resources, rotations/scales, invalid placement/permissions and multiplayer undo/redo.
- Exact executable/mod hashes, reproducible packaging and complete Steam/Cloud/guest isolation.

## Answered within scope

- Scripted local scenery create/move/delete and save/reload: 007–025.
- Independent equal-resource objects and guarded local replay: 026–030.
- Standard io/os/LuaSocket unavailable in the tested Lua; file roundtrip skipped: 031.
- Native arbitrary loopback destination and raw text response: 037b–047.
- Native HTTP usable with valid Kepler context without active Frontier connection: 043–045.
- Actual sequential A/B message exchange: 038; HTTP-triggered local CREATE/MOVE and duplicate skips: 040–042.
- Local full-body HTTP downloads up to 1 MiB: 046/047, not internet throughput proof.
- Shared-token grouped creation/commit up to 50 rocks: 049; one preparation Advance per group.

No playable co-op, simultaneous distributed state convergence or durable exactly-once guarantee is claimed.
