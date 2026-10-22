# Diagnostics 050–068: scale, recovery, history and profiling

**Reviewed 2026-10-22.** [Status](STATUS.md) · [Earlier ledger](DIAGNOSTICS_035_049.md) · [Measurements](../networking/BENCHMARKS.md) · [Recovery boundaries](../networking/RECOVERY_AND_HISTORY.md)

## Evidence method and scope

These are owner-run scripted diagnostics in PC2, not simultaneous two-client QA. Retained version-marked private game logs for 050–068 were reread for this update; earlier server screenshots/reviews corroborate the fault injection and journals. The documentation update did not run the game. A final result is accepted only when its counters/checks and completion agree. Raw logs, game resources, saves, identifying paths and checkpoint identifiers are not published.

Reported setup: Windows/Steam, PC2 **1.11.4.1011357**, ACSE **0.729**, ForgeUtils **2.7.0**, ACSEDebug **0.014**. Diagnostic 066 ran inside PC2 but used synthetic Lua records; its 100,000 entries are not 100,000 real scenery objects. 067/068 used real AQ_Rock_04 placements. No evidence here establishes arbitrary scenery support or playable co-op.

## Runtime results

| Diagnostic | Verified result | Limit / qualification |
| --- | --- | --- |
| 050 | 2,701 rocks; 13 previews/commits/checkpoints, 26 world-check passes; completed | Groups up to 500, one resource, no save; frame-dependent game clock hid same-Advance work |
| 051 | Same 2,701 rocks / 13 commits / 26 passes with minimal own logging; completed | Similar roughly 47 ms game-clock pipelines do not isolate logging CPU cost |
| 052 | Performance timer validated; same object/commit counts, 33,726 individual object checks; completed | Measures elapsed time, not CPU time; Advance gaps are not presented frame times |
| 053 | Only newly created groups checked twice: 5,402 object checks; 2,701 rocks / 13 commits; completed | Reduced verification workload; no reduction of native completion requirements |
| 054 | New-group checks plus final inventory: 8,103 object checks, 2,701 rocks / 13 commits; final resource/position/mapping/unique-primary checks passed | Final single inventory pass: 8.7688 ms work, 19.7844 ms elapsed; no save |
| 055 | Four HTTP deliveries: one DELETE accepted, one duplicate, one order error, one ID conflict; three rejection guards passed; four tasks joined | Setup CREATE plus DELETE gives two commits; no additional deletion on rejection |
| 056 | Local Create/Move/Delete with three native Undo and three Redo calls; 18 object checks; part restored under captured Part-ID and finally absent; completed | Three original commits/checkpoints; Entity-ID changed on recreation, stable within each two-Advance check |
| 057 | Corrected repeat: nine HTTP commands/tasks, three commits, three Undo, three Redo, 18 checks; completed | Earlier final-check failure from unbound Lua `next` and separate HTTP failure are failed runs, not passes |
| 058 | 27 deliveries/tasks: nine accepted, six duplicates, six order errors, six ID conflicts; 18 guards and 54 object checks; completed | Rejections preserved native counters, ledger/history state and world; same live client |
| 059 | Nine unique actions delivered 18 times; first ACK attempt for each returned 503 without storage; nine eventual ACKs, nine duplicate guards; all 37 tasks joined | Controlled application-level ACK refusal, not a TCP reset or loss after durable ACK storage |
| 060 | Corrected real connection-reset test: CREATE delivered twice, applied once; ACK retry stored; six tasks joined; completed | First run misclassified the native failure tuple; client stayed alive and retained its ledger |
| 061 | Real reset before first ACK storage for each of nine actions; 18 deliveries, nine duplicate skips, nine failures and nine eventual ACKs; 38 tasks joined, 72 checks; completed | Create/Move/Delete and Undo/Redo; three commits, three Undo and three Redo, no duplicated native actions |
| 062 | Server process exited after Move execution but before ACK storage; journal resumed at generation 2; nine unique actions, ten deliveries, one duplicate; 31 tasks joined, 40 checks; completed | One server restart; eight failed health probes then successful recovery; client process/ledger survived |
| 063 Phase 1 | One CREATE and MOVE to (15,5,10), two commits; exactly one new local save, server checkpoint sealed and manifest read back; eight tasks joined | Runtime application records became checkpoint-included only after successful save/checkpoint handling |
| 063 Phase 2 | Matching save loaded after client restart, fresh entity mapping read; two old commands skipped; only new MOVE to (20,5,10) committed; nine tasks joined; completed | Saved checkpoint remains at (15,5,10); new runtime move was not saved. Controlled completed checkpoint, not arbitrary crash recovery |
| 064 Phase 1 | Native Undo and Redo both available before and after one new local save; 12 checks; completed | Saving itself did not clear availability in this run |
| 064 Phase 2 | Same saved part at (10,5,10), fresh mapping, four checks; CanUndo=false and CanRedo=false stable over ten consecutive Advances; completed | Native history was unavailable after this tested restart. No Undo/Redo attempted; zero commits/saves. Completed means diagnostic completed, not history persistence passed |
| 065 Phase 1 | CREATE then MOVE; before/after coordinates recorded, one new local save, checkpoint string emitted; completed | Checkpoint transferred manually into Phase 2 source, not automatic persistent mod storage |
| 065 Phase 2 | Counteraction Undo moved 15→10; counteraction Redo moved 10→15; two new Move commits; two duplicates, two ID conflicts, one order error and one stale-version conflict rejected unchanged; 24 checks; completed | Zero native Undo/Redo calls and zero saves; object version 2→3→4. Only the tested positional Move, not general Delete reconstruction |
| 066 | All 12 object/history-size combinations and 24 layouts passed; equal paired classification/update results; simulated quarter-history pruning verified; completed | Synthetic records only; minimal/full maintain different extra data. Checkpoint pruning is a model, not durable journal compaction |
| 067 | 2,701 real registered rocks, 1,464 object moves in 72 groups; 84 commits/checkpoints; 180,370 world checks; all 84 entity groups and 168 native operations joined; completed | 108 probe samples; final positions/versions correct, no mapping changes. Some transition spikes lacked individual attribution |
| 068 | Corrected repeat retained 067 counts and checks; 6,867 callbacks paired with following gaps; exclusive section accounting passed with no section overflow; completed | Initial `scope` name-shadowing failure stopped at zero objects/commits. Detailed profiling adds overhead; only bounded chronological traces retained |

All successful 050–062 and 065 Phase 2 / 066–068 runs reported zero diagnostic save calls. Only expressly authorized Phase 1 runs of 063–065 created their respective new local saves. A native undo `Checkpoint({})` is not a park save.

## Identity and native history

`api.undo.Undo()` and `Redo()` take no explicit Lua arguments in the tested binding; returned operation tokens were awaited. Do not create an extra checkpoint between a native Undo and its matching Redo. Captured local Part-IDs survived the tested native recreation/reloads, but Entity-IDs changed: 063's old and freshly loaded entities differed, as did recreations in 056–062. Always refresh entity mappings; a local Part-ID is not a universal peer ID.

064's `originalHistoryAvailabilityRestored=true` means the test preserved the availability it found on loading. It does **not** mean the earlier session's native history was restored. The actual loaded flags were both false and both native-history verification flags were false.

065 restores the effect of one positional Move using new, version-checked counteractions. It does not rebuild the engine undo stack. Persisting a command's DONE status, an action's APPLIED/UNDONE state and the physical object checkpoint are separate concerns.

## Failure history and exact boundaries

- 057's earlier native work succeeded before its final Lua check failed; the later version-marked complete run is the passing result.
- 060 observed `nativeSuccess=false`, Kepler result 3, reported status field 2 and nil body after a real connection reset. That value 2 is not a valid HTTP response code. Do not accept arbitrary non-200 responses as transport failures.
- 059–061 deliberately refused/dropped the first ACK **before** storing it. Loss of the reply after durable ACK storage is a distinct case.
- 062 restarts the server, not the game client; 063 restarts the game after a completed saved checkpoint. Neither resolves a crash between native commit, durable command bookkeeping and saving.
- 068's corrected `waitSeconds` uses a parameter name distinct from its `scope` helper. Its first failed run produced no workload and is excluded from performance conclusions.

## Private evidence anchors

Logs are identified here by diagnostic and retained timestamp suffix only; they are not downloadable repository files. 050: 051345; 051: 074542; 052: 081007; 053: 082407; 054: 084100; 055: 092403; 056: 094837; corrected 057: 101719; 058: 102726; 059: 103847; corrected 060: 105654; 061: 110903; 062: 112129; matched 063 phases: 114931/115224; 064: 120922/121749; 065: 124518/125156; 066: 132305; 067: 133931; corrected 068: 140416, all dated 2026-10-22. Earlier failed 057/060/068 logs remain private as failure evidence.

Native resource and transform access observed in reconstructed PC2 sources: `Editors.Scenery.SubModeSelect:SetLastEntityAsSelection` uses `CreateBuildingPartSet`, `EntityIDToPlacementID`, `HasPresentedData`, `HasCommittedData`; selection paths use `PlacementIDToEntityID` / `PlacementIDToAllEntityIDs`. `SubModeBase:CommitPreview` polls completion and calls `Checkpoint({})`. Runtime 067 tests the actual simple-rock query path; source names alone do not establish general support.
