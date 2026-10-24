# Runtime tests — PC2 lifecycle, scenery and transport capability

[← Documentation guide](../README.md) · [Current status](../research/STATUS.md) · [Scenery research](../placement/README.md)

**Updated 2026-10-24, reviewed through 084 Step 11 and the own-logger test. Step 12 remains prepared/unverified.** This page separates experiments that **ran in PC2**, tests that were **not run**, and historical failures. Log excerpts here are short manually transcribed observations; **complete private logs are not committed**.

## Latest completed gates — 083/084

[Detailed evidence ledger](../research/DIAGNOSTICS_083_084.md) · [Native bridge, measurements and limits](../networking/NATIVE_BRIDGE.md).

| Test | Runtime result | Limit |
| --- | --- | --- |
| 083, four spacing runs | 220 correct probes and 224 joined tasks each; one-second tails at 50/250 ms; fast samples restored after a 120-second pause | Internal limiter/quotas hypothetical, 500/1,000-ms runs finite |
| 084 Steps 6/7 | Direct Lua input/output strings with build/stack/result checks and invalid-input rejection | Exact tested native profile, not arbitrary Lua ABI |
| Step 8 | 4,100 round trips, 13 full GCs/control canaries, 6,662 retained-answer checks; both completion markers true | No universal memory/GC proof |
| Steps 9/10 | 1,010 messages plus three Python notices each; one-sandbox then sandbox-to-external-Python pipe; all completions | Same machine |
| Own logger, both modes | File-content review: 1,000 exact test lines / 1,014 queued records each; no losses or errors; both completion markers true | Owner-disabled ACSEDebug, not auto-detected; logger is not full engine debugger |
| Step 11 | 1,000 messages each direction verified and acknowledged; own logging; both Lua/native pairs and external Python complete | Message-only local relay; no game/save mutations |
| Step 12 | Code and instructions supplied for manual Create, next-preview continuity and duplicate application guard | **Not a passed test yet** |

Step 11 held one A-to-B message for 2,001.2016 ms while 112 B-to-A messages were acknowledged. A/B observed 146/175 Advances during the delay, with max gaps 22.6718/13.3105 ms. These are Advance observations, not rendered-frame telemetry. The earlier HTTP scenery/recovery results do not automatically apply to this native channel.

The owner reports a recurring crash reporter at game exit, including the reported setup without ACSEDebug. The cause has not been isolated. Successful diagnostic/channel completion must not be described as proof of a crash-free process shutdown.

## Earlier completed gates — 069–082

The earlier tables retain their historical scope. [069–082 ledger](../research/DIAGNOSTICS_069_082.md) contains the later per-test outcomes and corrected/aborted-run distinctions.

| Diagnostic | Runtime result | Important limit |
| --- | --- | --- |
| Later 038 repeat | Two local clients send/receive three numbered messages each and complete | Same-machine loopback, not internet |
| 069/070 | Split full-result materialization/search; 110 targeted known/unknown queries pass | One known scenery resource, no world edits |
| 071–073 | Scripted Create, Move to (15,5,10), Delete and duplicates pass on both clients | Not manual player Move/Delete capture |
| Corrected 074 | Nine unique actions, 18 deliveries, three commits plus three Undo/Redo each per client | Native live-session stacks |
| 075/076 | Nine refused first ACKs / TCP aborts per client, replay without duplicate effects | ACK not stored before injected failure |
| 077 | Controlled resumed server restart, same live clients, nine unique effects each | Not arbitrary client crash |
| 078 | B saves once/restarts while A lives; restored records/mappings and new Move only | Explicit completed save/checkpoint pairing |
| Corrected 079 | Actual player-confirmed host Create captured and applied once on B | Corrected test waits for placement-tool exit |
| 080/081 | Same manual Create with distinct active next preview kept local | Sampled preview state, not every rendered frame |
| Completed 082 | 14 phases, all 1,200 probes correct; A/B/P/server complete, all native tasks joined | Exact Kepler delay cause still unresolved |

082's final task counts are A 552/552 and B 646/646. Guest Python runs in Sandboxie and performs direct TCP/HTTP measurements. No database/editor/save changes are made. Earlier aborted 082 reports on world deactivation are not completed runs; all participants must remain alive through phase 14 and final acknowledgements. [Measured results](../networking/HTTP_LATENCY_080_082.md).

## Environment and isolation boundary

| Property | Observed or reported value |
| --- | --- |
| PC2 game build | `1.11.4.1011357` (self-reported at log startup) |
| ACSE | `0.729` |
| ForgeUtils | `2.7.0` |
| ACSEDebug | Historical tests used `0.014`; own-logger and Step-11 runs were performed without it per owner setup; interactive GUI compatibility remains unproven |
| Diagnostic mod | `PC2CoopDiagnostics`, Lua code packed as `Main.ovl` |
| Game execution for diagnostics 005 and 006 | **Separate copied test installation, but *not* launched through Sandboxie**, per tester confirmation |
| Game / mod binary SHA-256 | Not published / not established in these records |

The existence of an already configured Sandboxie `PC2CoopHost` environment is a **separate** setup fact. Do not claim the successful 005/006 game sessions were sandboxed. See [isolation setup](../ISOLATED-INSTALLATION.md).

## Chronological test record

| Test / private log | What changed | What was actually observed | Interpretation |
| --- | --- | --- | --- |
| Initial component test — `Planet Coaster 2(1).log` | Enabled custom `AddLuaComponents` diagnostic with editor availability probes and **no explicit `Shutdown()`** | Component initialized and menu-world method checks were true; park transition stopped after old-world shutdown | **Failure.** Underlying cause unknown; no native exception trace. |
| A/B control — `Planet Coaster 2(2).log` | Disabled **only** custom component registration while retaining ACSEDebug and `LuaDB.Init()` | Tropical park loaded, ran over 60 seconds and shut down normally | Component involvement strongly suggested; does **not** prove `Shutdown()` alone fixed it. |
| Packaging controls — `Planet Coaster 2(3).log`, `(4).log` | Intended minimal world-component test, but earlier `Main.ovl` still in use | `LuaDB.Init()` logged but custom component was **not registered** | **Not** valid evidence for the new component. |
| **Diagnostic 005 — `Planet Coaster 2(5).log`** | Added `VERSION TEST 005`; rebuilt/copied the correct `Main.ovl`; registered minimal component with `Shutdown()` | New version marker, registration, `Configure`, `Init`, and `OnWorldActivation` appeared in menu world | Confirmed packaging and basic component lifecycle. |
| **Lifecycle / park transition — `Planet Coaster 2(6).log`** | Continued with the registered minimal component | Menu-world `OnWorldDeactivation` and `Shutdown`; new component registered/configured/initialized in park; `tropicalworld` activated; park completed load and normal exit | **Successful** park-world component lifecycle. Tester also changed screen mode while running. |
| **Diagnostic 006 — `Planet Coaster 2(7).log`** | Added guarded, presence-only checks on world activation | In `tropicalworld`: `GetWorldAPIs` succeeded; `editors` accessible; `CreateUndoOperationsHierarchy` and four `api.undo` methods present as functions. Park finished loading and later deactivated normally. | **Successful API presence test**, not a method-invocation or scenery-edit test. |
| **Diagnostic 007** | Invoked hierarchy construction in the park | Returned hierarchy used successfully in subsequent tests | Passed; historical planned status superseded. |

### Key runtime checkpoint: diagnostic 006

In the active `tropicalworld`, the log reports:

```text
[PC2-COOP] Current world: tropicalworld
[PC2-COOP] GetWorldAPIs succeeded: true
[PC2-COOP] Editors API accessible: true
[PC2-COOP] CreateUndoOperationsHierarchy available: true
[PC2-COOP] Undo API accessible: true
[PC2-COOP] PreviewChangeList available: true
[PC2-COOP] CanCommitPreview available: true
[PC2-COOP] CommitPreview available: true
[PC2-COOP] CancelPreview available: true
[PC2-COOP] EDITOR DIAGNOSTICS 006 COMPLETE
```

These entries are **presence checks only**. `pcall()` around a Lua call is not a guarantee against a native engine crash.

## Diagnostics 007–034: later runtime results

The detailed [diagnostic ledger](../research/DIAGNOSTICS_007_034.md) records each step and its limits. These owner-run tests are not a two-client automated QA suite.

| Range | Result |
| --- | --- |
| 007–013 | Hierarchy/change list, CreateNewPart, StartEditingPart_Scenery, transform reads and reversible cross-Advance movement |
| 014–018 | Completed preview, cancellation, committability, one commit and world-instance verification |
| 019–022 | Creation and existing-part movement persisted through new local saves and manual verify-only reloads |
| 023–025 | Deletion preview restored on cancellation; committed deletion persisted after save/reload |
| 026–027 | Two equal-resource rocks addressed independently by own keys/local IDs |
| 028 | Eight commands, four duplicate skips, four commits |
| 029 | Four order errors and four ID conflicts rejected; four valid commits |
| 030 | A/B separate runs: 120 matching canonical STATE records, covering all twelve inputs |
| 031 | io/os/LuaSocket unavailable; file/socket tests skipped; loadlib present but untested |
| 032–034 | Raw enumeration incomplete; five normal-access controls function; nine invented names nil |

### Retained evidence rechecked during this update

- 022 Phase 1: CommitPreview call count 1, operation completed, save returned success; Phase 2 checks the loaded part/resource/position without mutation.
- 025 Phase 1: one completed deletion commit and successful save; Phase 2 checks absent part and mappings.
- 030 A/B: twelve verified snapshots, each ending with four applied and eight rejected inputs; canonical comparison excludes instance labels/timestamps and local IDs.
- 034: checked=14, controlFunctions=5/5, inventedFunctions=0/9, inventedNil=9, failedOrSkipped=0, Completed=true.

No tests were rerun in the game for this documentation update. Earlier results are consolidated from prior private log reviews; retained selected logs were reread.

## Diagnostics 035–049: native transport and grouped edits

See the [detailed evidence ledger](../research/DIAGNOSTICS_035_049.md), [native request contract](../networking/TRANSPORT.md) and [measurements](../networking/BENCHMARKS.md).

| Range | Result and limit |
| --- | --- |
| 035b / 036 controls | Lua-JS roundtrip passed; JS external request did not reach server; independent same-sandbox Python loopback passed |
| 037 / 037b | Early cancellation; later native GET HTTP 200 / text response / task join after initialization |
| 038 | Sequential A/B/A-again three-message exchange; retained server recognizes repeat A sends; no simultaneous pair |
| 039 | Sixteen downloaded validation inputs: four accepted, four each duplicate/order/conflict, no park edits |
| 040–042 | Native HTTP CREATE/MOVE and duplicate suppression, correct positions and single intended commits |
| 043–045 | Loopback survives Frontier loss; offline-start requests succeed after Kepler user context exists; 043 recovery incomplete |
| 046/047 | Corrected 046 online/offline 27 each; 047 110 correct full bodies and joined tasks; local single-request timing only |
| 048/049 | 274 unique parts, 16 commits/checkpoints, 32 world checks; shared wait reduces up-to-50 creates to one preparation Advance |
| 050–054 | 2,701 rocks / 13 commits, performance timer and reduced verification; final 054 inventory passed |
| 055–058 | HTTP DELETE and native live-session Undo/Redo; duplicate/order/ID-conflict guards passed |
| 059–062 | ACK refusal, real TCP resets and journal-backed server restart; live client avoided duplicate actions and joined tasks |
| 063 | Controlled completed-checkpoint client restart: two old commands skipped; only new Move committed |
| 064/065 | Native history absent after tested restart; manually transferred Move data supported new counteractions and conflict guards |
| 066 | All 12 synthetic size combinations / 24 layouts passed; no world mutation |
| 067/068 | 2,701 real registered rocks, 72 Move groups, 84 commits, 180,370 checks; corrected 068 profiling and completion passed |

This documentation update does not run the game. Older overwritten logs are summarized from prior reviews, not claimed as newly re-executed tests. Retained 038 logs and 047 source/server output were reread; mocks are excluded from runtime evidence.

## Next experiment and unproven scope

Simultaneous local messaging, scripted events and one ordinary host Create have now passed in 071–081. 083 has now tested request spacing/recovery and 084 supplies the preferred own local bridge. Next validate Step 12 game-effect replication, pipe recovery and exit behavior, then extend manual Move/Delete and a secured remote route. Earlier local recovery/benchmark outcomes through 068 are in the [050–068 ledger](../research/DIAGNOSTICS_050_068.md); later evidence is in [069–082](../research/DIAGNOSTICS_069_082.md). [Proposed protocol](../networking/HTTP_SYNC_DESIGN.md).

No general free-editing/internet gameplay, broad player-edit capture, automatic durable deduplication, distributed reconnect/resync or full simulation synchronization is established. Specific local scripted edits and one HTTP manual Create remain verified as described above. 062 tests server restart with a surviving client ledger; 063 tests client reload from a completed saved checkpoint, not arbitrary crash recovery. Full Steam/Cloud isolation remains unvalidated; later successful local Guest runs establish only their tested arrangement.

## Evidence handling

Keep unredacted logs, absolute Windows user paths, game executable hashes, game binaries, mod OVL files and saves **locally**, not in this public repository. See the [evidence index](../research/EVIDENCE.md) and [chronological research log](../research/LOG.md).

## Latest evidence and superseded failures

The [050–068 ledger](../research/DIAGNOSTICS_050_068.md) gives per-test counts, private evidence anchors and boundaries. Failed early 057/060/068 runs remain failures; corrected repeats alone support completion claims. 064 completed successfully as a diagnostic **finding native history unavailable**, not as proof that it persisted. 065's action data were manually transferred. No game was run for this documentation update.
