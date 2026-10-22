# Diagnostics 035–049: transport, command replay and batching

**Consolidated 2026-10-22; historical scope 035–049. Later reviewed outcomes: [050–068](DIAGNOSTICS_050_068.md).** [Earlier ledger](DIAGNOSTICS_007_034.md) · [Status](STATUS.md) · [Native HTTP](../networking/TRANSPORT.md) · [Benchmarks](../networking/BENCHMARKS.md)

## Evidence method

Owner-run PC2 tests, reviewed in the ongoing research session. This update consolidates prior log/screenshot reviews; it does not claim every older overwritten log is still available or was newly rerun. Retained 038 A/B/A2 logs, the 047 source/server log and previously reviewed 046–049 outcomes anchor the transport/performance account. Sources are private diagnostic components, ACSEDebug logs and local Python server output. No raw logs, game code, OVLs or identifying machine paths are published.

Reported environment: Steam/Windows PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0, ACSEDebug 0.014. Exact executable/mod hashes are not established publicly. Packaging mistakes produced stale/no-op runs at several earlier steps; only version-marked successful repeats count. No game session was started by this documentation update.

## Runtime findings

| Diagnostic | Observed result | Boundary |
| --- | --- | --- |
| 035 / 035b | Initial view/package-loading investigation; corrected 035b completed custom Lua -> JavaScript -> Lua message roundtrip and view/resource cleanup | Embedded UI bridge, not network transport |
| 036 / 036b | JS request reported 404; server saw no matching HTTP request; URL/body diagnostics added | 404 is not evidence our server route was reached |
| 036 loopback control | Separate Python process in PC2CoopHost reached same loopback URL with HTTP 200 and correct response marker/body; host control also passed | Demonstrates that control process's path, not all game process permissions |
| Native Gameface inspection | Prior private resource-handler investigation consistent with local resource dispatch/non-forwarding for the tested external request | Static investigation, not a universal statement about Gameface |
| 037 | Early native request failed with Kepler code 19, server saw no request | 19 identified as Cancelled by prior native enum inspection; failure cause not determined solely from code |
| 037b | After full online initialization, one native GET reached server, returned HTTP 200 / expected sBody, task joined | Native text transport established; full Frontier gate later relaxed |
| 038 | A sent three messages; B later sent three and read A's three; A-again read B's three; B and A-again completed, repeat A sends recognized as server duplicates | Sequential game runs with retained in-memory server, NOT two simultaneous clients; initial A exhausted its wait before B started |
| 038 internet preparation | Internet relay/client instructions prepared | Remote friend test not completed; HTTPS/auth deployment not validated |
| 039 | 16 downloaded inputs: 4 accepted, 4 duplicates, 4 order errors, 4 ID conflicts; rejected inputs left validation state unchanged, tasks completed | Validation/shadow state, no park edits |
| 040 | One downloaded CREATE applied/committed once; world position (10,5,10) verified across two Advances; tasks completed | Single local client driven by server; no save |
| 041 | Same CREATE redelivered over HTTP skipped; create and commit counts remained one, world unchanged | Session-local dedup only; no crash/reload guarantee |
| 042 | Downloaded CREATE, MOVE and duplicate MOVE: two accepted, one duplicate, three joined HTTP tasks; Create committed once and Move committed once; position (15,5,10) stable | No HTTP DELETE world execution test here; earlier local delete tests are separate |
| 043 | 39 joined HTTP tasks: 2 baseline, 34 watch, 3 loss; local requests passed after Frontier disconnect with Kepler context retained | Recovery did not finish / overall Completed=false; do not call complete recovery verified |
| 044 | Internet disconnected before start; early no-context request failed; later two requests HTTP 200 after Kepler user context appeared, no Frontier connection | No proof of successful HTTP with no Kepler user context or fresh uncached profile |
| 045 | Gate only valid owner + Kepler available + Kepler user; all three requests succeeded while Frontier disconnected, tasks completed | Supports relaxed readiness gate; Steam/service-independent cold bootstrap not proven |
| 046 | Valid online and corrected offline runs each passed 27 downloads, full content and task joins, 1 KiB–1 MiB | Ascending order/startup timing confound; an earlier mislabelled offline file was excluded |
| 047 | 110/110 requests/full bodies/task joins/reference releases; 10 warmups + 100 mixed measurements; server sizes/sequences matched | Local, one request in flight, no uploads/editor actions/concurrent clients |
| 048 | 274 unique rocks, 16 group commits/checkpoints, 32 verified world checks; preparation grew by one Advance per rock | Sequential create/wait per object; no save |
| 049 | Same 274 parts / 16 commits / 32 checks; all requests issued before shared-token wait; one preparation Advance per group, four pipeline Advances | Grouped creation tested to 50; no large-park/internet proof |
| 050 handoff | Originally pending at this ledger cutoff; subsequently passed and reread | See the [050–068 ledger](DIAGNOSTICS_050_068.md) for reviewed results |

## Concrete source and symbol anchors

- Original diagnostic 047 component: readiness `readStatus` (GetGameOwner / IsValid / IsKeplerAvailable / IsKeplerUser); spawned worker (CreateKeplerHttpRequest, SetUrl, SetMethod, SetGenerateAuthCode, RunKeplerRequest third response); Advance JoinAsync handling. Its retained source corroborates the exact generic native contract.
- Original 038 diagnostic `RESULT` / `Completed` records and corresponding Python server MESSAGE/BATCH/duplicate outputs: distinguish sending, receipt, server duplicate handling and actual simultaneous execution.
- Original 048/049 `runBatch`, PHASE_BEGIN/END, SAMPLE, MAPPING, WORLD_CHECK, RESULT and CoroutineFinished records: group identity, native call counters, positions, task-flow completion and measurement boundaries.
- Original 046/047 expected-body generators and complete string comparisons: body length alone was not the integrity check. The mock QA logs in source packages are synthetic and are excluded from real runtime/performance evidence.
- Exact generic HTTP source excerpt and limitations: [TRANSPORT.md](../networking/TRANSPORT.md). Numeric measurements: [BENCHMARKS.md](../networking/BENCHMARKS.md).

## What remains unproven

No simultaneous two-client scenery replication, actual internet test, normal player-edit capture, durable exactly-once effects, persistent distributed identity, reconnect/resync, full park simulation, or playable mod. Hypothesized [HTTP synchronization architecture](../networking/HTTP_SYNC_DESIGN.md) is deliberately separate from these observations.
