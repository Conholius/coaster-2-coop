# Evidence index

These are names of locally researched materials, **not bundled source files**. The original copies remain under ignored `local/` or a separately maintained authorized evidence archive. The dates/builds below are file-label claims, not necessarily the version of the installed executable. The first table and 005/006 entries preserve their historical evidence scope; current runtime results through 084 Step 11 are indexed in the later sections and in [STATUS.md](STATUS.md).

| Area | Local source material | Confirmed symbols/observations | Qualification |
| --- | --- | --- | --- |
| Native API inventory | `PC2_1.0.6.429191_API.lua` | `UndoOperationsHierarchy`, `Placement`, `Undo` API symbol names | Names only; signatures and behavior unproven |
| High-level game | `game.basegame_unluac.lua`, `game.gamescript_unluac.lua`, `game.gameenvironment_unluac.lua` | API startup, world/environment setup, optional debug commands | Decompiled; check fragile syntax |
| Lua module inspection | `api.debug_unluac.lua`, `common.moduleutils_unluac.lua` | `package.loaded` enumeration, `loadresource` reload; module metatable checks | Module-to-OVL mapping unknown; debug gated |
| Editor registration | `gameeditormodes_unluac.lua`, `gameeditoruimodes_unluac.lua` | Logical editor and UI mode names | Logical module names need not equal OVL resource names |
| Runtime worlds | `world.world_reconstructed.lua`, `environment.environment_unluac.lua` | world lifecycle and manager phase sequencing | World-specific park environment not fully located |
| Object creation | `commonobjectcreator_unluac.lua` (Common archive) | `InstantiatePrefab`, transforms, parent entity | Not evidence of durable placement |
| Save orchestration | `managers.parkloadsavemanager_unluac.lua` | `RequestSaveWorldToFile`, `HasSaveToFileCompleted`, `DoPendingParkLoad` | Serializer implementation not present |
| Online local save versions | `managers.localchangemanager_unluac.lua` | save tokens, online park IDs, version numbers | Not an editor change-event stream |
| Scenery components | `scenerypuppet.specdef`, `partdata.specdef`, `partdataresourcename.specdef`, scenery archives | placement part ID and resource fields | Struct definitions not runtime call traces |
| Serialization components | `standalonesceneryserialisation.specdef`, `voxelterrainserialisation.specdef`, `guestserialisation.specdef`, `staffserialisation.specdef`, `worldserialisationtest*.specdef` | Prefab and transform references, other component data | Partial serialization schema only |
| Mod comparisons | Enhanced Move Tool; ProTrack; Frontier modding references | Lua editing and track editor hook patterns | Cross-game methods must not be treated as PC2 proof |
| Scenery editing (2026-10-20) | `editors.scenery.rar`, `pathsnap.lua`, `draggablewidgets.lua`, `axispositioning.lua`, `translationgizmo.lua`; original editor `.lua.bin` | `CreateUndoOperationsHierarchy`, `CreateNewPart`, `StartEditingPart_Scenery`, `StartPreview`, `CanCommitPreview`, deferred `CommitPreview`, snapping and gizmo transforms | Decompiled call sites; runtime, persistence and external mod access untested; vector placeholders remain |
| Bytecode inspection | original `.lua.bin`, converted headers and `unluac` disassembly | PC2 Lua 5.3 custom format; conversion works for many files | Some decompilations fail despite valid disassembly |

## Selected anchors in reconstructed files

- `managers.parkloadsavemanager_unluac.lua`: `_RequestSaveParkToFile`, `GenerateCurrentParkMetaData`, `DoPendingParkLoad`.
- `managers.localchangemanager_unluac.lua`: `_CacheSaveIfLocalChange`, `_FindLocalChanges`, `DeleteOutdatedLocalChanges`.
- `api.debug_unluac.lua`: `API_GetListOfLoadedModuleNames`, `API_ReloadModule`.
- `common.moduleutils_unluac.lua`: `DisallowIndexNewIndexExcept` and metatable guards.

**Next evidence improvement:** attach stable source build identification, reproducible extraction steps and hashes of original artifacts in private research records (not proprietary contents). 

## Public source follow-up (2026-10-20)

See [ACSE / ForgeUtils investigation](../placement/ACSE_FORGEUTILS.md) for commit-pinned source references and function-level anchors. ForgeUtils catalogue registration, its Lua hooks, ACSE's tracked-entity serializer and PC-CC's track preview/commit implementation were inspected. These are static source observations; none establishes PC2 scenery save/reload persistence.

## Runtime evidence (2026-10-21; private PC2 logs)

All successful 005/006 sessions in the following table used a **copied game installation outside Sandboxie**, as confirmed by the tester. Do not imply the runs were inside the separately configured Sandboxie host.

| Test / source | Verified in a running game | Not established |
| --- | --- | --- |
| Baseline diagnostics; `Planet Coaster 2.log` | ACSE `0.729`, ACSEDebug `0.014`, ForgeUtils `2.7.0`; `LuaDB.Init` and `api.debug.Trace` work; `forgeutils.hookmanager` can be required and has an `AddHook` function | No hook execution or cross-game debug GUI compatibility |
| Original crash; `Planet Coaster 2(1).log` | Custom component registers, initializes and probes menu-world API presence; crash/termination around old-world shutdown during attempted park transition | Native crash cause; cannot attribute it solely to missing `Shutdown` |
| Component-disabled control; `Planet Coaster 2(2).log` | With custom component registration disabled but ACSEDebug enabled, a tropical park loads, stays up over 60 seconds and exits normally | Cannot isolate which custom component feature caused initial crash |
| Stale-package runs; `Planet Coaster 2(3).log`, `(4).log` | Diagnostic `LuaDB.Init` ran, park loaded; **the custom component was not registered** | Not evidence of minimal-component safety |
| **Diagnostic 005; `Planet Coaster 2(5).log`** | `VERSION TEST 005` appeared; custom component registered; `Configure`, `Init` and menu activation executed | Park-world lifecycle not yet tested in that log |
| **Minimal component park transition; `Planet Coaster 2(6).log`** | Old world deactivation and `Shutdown`; component recreated and activated in `tropicalworld`; park loaded and exited; screen-mode change logged | `Shutdown` alone not isolated as root-cause fix |
| **Diagnostic 006; `Planet Coaster 2(7).log`** | In `tropicalworld`, `GetWorldAPIs` succeeded; `editors` and `api.undo` accessible; `CreateUndoOperationsHierarchy`, `PreviewChangeList`, `CanCommitPreview`, `CommitPreview`, `CancelPreview` present as functions; park ran and exited normally | **No** undo hierarchy created, no edit executed, no commit, no saved placement |
| **Diagnostic 007 (later passed)** | Hierarchy construction and subsequent use confirmed in later private log reviews | See the consolidated 007–034 record below |

The detailed experiment matrix is [runtime tests](../testing/RUNTIME_TESTS.md). Original logs are **private and not stored in GitHub**; source paths, user names, dumps, save files and binary packages remain in local evidence storage. Build **1.11.4.1011357** is game/log-reported; executable and OVL SHA-256 values are not recorded in these public notes.

## Historical initial Sandboxie setup (not the above 005/006 runs)

The project setup notes describe `PC2CoopHost` and an unvalidated `PC2CoopGuest`, file-redirection rules, and a startup configuration with `UnstrippedToken=y` allowing a main-menu launch in the host. This is a **reported setup / past startup observation**, not proof that 005/006 were executed in Sandboxie, that the guest launches, that Steam Cloud is fully isolated, or that Denuvo caused the original failure. See [translated setup notes](../ISOLATED-INSTALLATION.md).

## Later runtime evidence — diagnostics 007–034

The [detailed ledger](DIAGNOSTICS_007_034.md) distinguishes original test outcomes from this update's rereads. Older standalone logs are not all retained in the currently available backup folders; their conclusions are consolidated from the earlier research conversation and log reviews, not represented as newly repeated tests.

| Evidence anchor | Finding | Limit |
| --- | --- | --- |
| Prior private log reviews 007–018 | Hierarchy/create/edit/transform, preview/cancel/commit and world-position checks | One scenery resource and controlled scripted tests |
| 019 Phase 1 and prior Phase 2 review | New local save; AQ_Rock_04 / part 1182 at (10,5,10) after reload | One park's part ID |
| Retained 022 Phase 1/2 logs | One move commit, successful save, stable loaded resource/position (15,5,10) | Not cross-client mapping |
| Retained 025 Phase 1/2 logs | One delete commit, successful save, absent part/entity mappings after reload | Tested save only |
| Prior reviews 026–029 | Independent equal-resource parts; duplicate/order/ID-conflict checks | In-memory session state, not crash-safe exactly-once |
| Retained 030 A/B logs | Twelve snapshots / 120 canonical STATE records per run matched; four commits | Separate local runs; no network |
| Prior reviews 031–033 | io/os/socket unavailable; raw API enumeration incomplete; five candidate fields function | No free URL / response / bridge proof |
| Current 034 log | Five controls function, nine invented fields nil; checked 14, failures 0, completed true | No candidate function invoked |
| Original FrontierAccountManager bytecode and full private disassembly | 135 prototypes including main; RunKeplerRequest call corroborated at original line 1772 | No CreateKeplerHttpRequest call found |
| Complete private --nodebug decompile | 134 declarations plus main through file end; synthetic local names | Not runtime-equivalence proof |

Source/function anchors for the workshop, account, newsfeed, WebRequestManager and online-session investigation are in [TRANSPORT.md](../networking/TRANSPORT.md). Static findings and function existence must not be promoted to executed transport.

The first 005/006 records retain their outside-Sandboxie qualification. Later log paths do not independently establish process isolation. No raw logs, decompiled scripts or private Windows paths are committed.


## Network and batching evidence — 2026-10-22, diagnostics 035–049

See the [035–049 ledger](DIAGNOSTICS_035_049.md) for per-diagnostic outcomes, sources and boundaries. These are owner-run tests and prior log reviews, not newly executed game sessions in this update.

| Source anchor | Finding | Boundary |
| --- | --- | --- |
| Prior 035b/036 reviews and owner server/control screenshots | Lua-JS roundtrip; JS 404 without server hit; same-sandbox Python loopback success | UI bridge and control path, not external JS HTTP success |
| Prior native Gameface / Kepler inspection | Resource-path explanation for tested JS behavior; result 19 identified as Cancelled | Static/private investigation; no general Gameface or cancellation-cause guarantee |
| Retained original 047 worker/readStatus/Advance source | CreateKeplerHttpRequest(playerID), setters, third response table nResponseCode/sBody, Spawn/JoinAsync and relaxed context gate | Original mod code inspected, not redistributed game source; no remote HTTPS/POST proof |
| Retained 038 A/B/A2 logs and prior server screenshot review | Actual sequential three-message exchange; server duplicate detection on A-again | No simultaneous client pair; no crash-durable relay |
| Prior 039–042 log/server reviews | Validation counts; HTTP local CREATE/MOVE and duplicate suppression | No HTTP DELETE application or distributed exactly-once guarantee |
| Prior 043–045 reviews | Local requests after Frontier loss / offline start with ready Kepler context | 043 recovery incomplete; no no-context/no-cache/Steam-outage proof |
| Correct 046 online/timestamped offline reviews; prior 047 client review and retained server/source | Full bodies/tasks passed; 27 per valid 046 run, 110 in 047; bounded local latency | Excludes mislabelled 043 log and all mock logs; no internet/concurrency throughput proof |
| Reviewed real 048/049 MAPPING/WORLD_CHECK/PHASE/SAMPLE/RESULT records | 274 unique parts, 16 commits/checkpoints and 32 world checks in each; shared completion reduces Advances | Up to 50 per group, one rock resource, no saves; game-clock and trace limitations recorded |
| Supplied 050 code and owner's current test intent | Planned 2,701 rocks / 13 commits / 26 checks | No reviewed result; no performance conclusion yet |

The [HTTP synchronization design](../networking/HTTP_SYNC_DESIGN.md) is labelled proposed. Only [observed measurements](../networking/BENCHMARKS.md) are described as passed. Private raw evidence, proprietary archives and account details remain outside commits.

## Scale, recovery and profiling — diagnostics 050–068, reviewed 2026-10-22

The retained private version-marked logs were reread for this update. [Per-test outcomes and timestamp anchors](DIAGNOSTICS_050_068.md). No game was run and no raw logs/saves/dumps were published.

| Evidence | Finding | Boundary |
| --- | --- | --- |
| 050–054 logs and own diagnostic timing/check logic | 2,701 rocks / 13 commits, timer validation, 33,726→5,402 checks then 8,103 with final audit | One resource; frame clock versus performance timer; not a universal frame budget |
| 055–058 successful logs | HTTP DELETE/native Undo/Redo, immutable duplicate/order/ID conflict guards, refreshed entities | Session-local, no automatic global history |
| 059–061 logs plus previously reviewed server fault output | 503 ACK refusal and TCP-reset-before-ACK-storage redelivery without duplicate native effects | Live-client ledger; loss after stored ACK still distinct |
| 062 game log and prior server journal/restart review | Changed server generation/boot and resumed cursor; one duplicate skipped, 31 tasks joined | Server restart only, not client-crash atomicity |
| Matched 063 Phase 1/2 logs and prior server manifest review | One new local saved checkpoint, two restored records, fresh entity mapping, only new Move committed | Controlled completed checkpoint; unsaved runtime state not folded into saved checkpoint |
| 064 Phase 1/2 | Undo/Redo true after save, false after tested restart; loaded object state correct | Diagnostic completion is not native-history persistence success |
| 065 Phase 1/2 | Explicit positional Move data and two new counteractions; revision/ID/order/duplicate guards | Manual checkpoint transfer; no native-stack reconstruction or automatic mod storage |
| 066 | 24 synthetic minimal/full variants passed, 100,000-record scales | Not 100,000 physical scenery objects or isolated memory attribution |
| 067 | Real registry/native queries, 72 Move groups, 180,370 checks | Transition maxima aggregated; not paired causal evidence |
| Corrected 068 | 6,867 paired callbacks, exclusive accounting, 37.23 ms catalogue section, WORK callback/gap maxima 3.47/21.68 ms | Additional profiler overhead; catalogue materialization/search not separated; no retrospective explanation of 067 outliers |
| ACSE public source at 8a843e61bcb57779778536e635d334d882cee7b1, component manager Init / WorldSerialisationClient_Save / Load / Shutdown | Registered world-save/load callbacks and SaveEntityID/LoadEntityID mapping | Installed load callback observed; custom-registry integration and source/package identity still unproven |
| ForgeUtils moddb.lua at 90b78c31693a473e73d621af145053c3b5fc8059 | Mod/version registration, not instance/action storage | No native replacement for all synchronization semantics found in inspected material |

Source-function links are in [ACSE/ForgeUtils](../placement/ACSE_FORGEUTILS.md) and [recovery/history](../networking/RECOVERY_AND_HISTORY.md). The supplied API inventory is labelled 1.0.6.429191 and prefab dumps include 1.10.2.906765; current runtime logs identify 1.11.4.1011357. Presence in an older inventory is not sufficient behavior evidence for the current build.

## Two-client and latency evidence — 069–082, reviewed 2026-10-23

[Detailed ledger](DIAGNOSTICS_069_082.md) · [Timing definitions and full 082 phase table](../networking/HTTP_LATENCY_080_082.md). Retained logs were read; no game or diagnostic was run for this publication.

| Evidence anchor | Verified or observed finding | Boundary |
| --- | --- | --- |
| 069 FIRST/REPEAT paired samples and RESULT | 7,263 rows, 32.5976 ms retrieval, 4.2579 ms first scan; twenty retained-table scans average 3.9730 ms | Not guaranteed cold; elapsed not CPU; Advance gaps not presented frames |
| 070 per-statement summaries, CALLBACK_TOTAL and RESULT | 110 known/unknown targeted queries; no full catalogue; own max 0.1188 ms, gap max 18.5428 ms | Exact known resource and statement semantics only |
| Later 038 Host/Guest RESULT and completion | sent=received=3 each, matched fresh session, 47/4 tasks | Earlier 038 sequential scope retained; no internet test |
| 071–074 paired client RESULT/world records | Both-client scripted scenery and native history with duplicate suppression | One resource/controlled order, not free simultaneous editing |
| 075/076 paired RESULT plus original fault-injection server logic/prior reviews | Nine refused/dropped first ACKs and nine duplicates per client, no additional native effects | Before ACK storage; 076 server cleanup fix separate |
| 077 paired RESULT and journal/generation server code | Resumed generation 2, one duplicate and nine unique actions each, A/B 98/70 tasks joined | Controlled process restart only |
| 078 A/B Phase 1/B Phase 2 RESULT and checkpoint/restore records | A stays alive; one B local save, two restored records, fresh entity lookup, only new Move committed | Successful complete checkpoint, not arbitrary unsaved crash |
| Corrected 079 hook/capture/world/RESULT records | Hook actually invoked, one original manual commit/capture/publication, Guest one effect plus duplicate skip | Registration alone not invocation; guest capture disabled |
| 080 matched A/B results and server proofs | Distinct committed/preview parts, 45/45 active observations, one Guest Create/commit and duplicate skip, tasks 53/69 | Next preview local, no per-presented-frame proof |
| 081 events/measurements and server summary | B commit call/wait 0.017/16.231 ms; 136.577 ms receipt-to-commit; A proof latency 2239.214 ms | Independent clocks; no logging-only controlled comparison |
| Completed 082 A/B, server and Guest Python records | 14 phases; 480/600/120 probes; all 1200 valid; A/B tasks 552/646 joined; no unmatched/open probe connections | Loopback, Python 3.11.0, local clocks only, original HTTP/1.0 connection-close path |
| 082 request/server/control timing records | Long waits inside native Run; fast accepted-handler/write and Python controls; larger queue/ports insufficient | TCP pre-accept vs post-response native return not separated; fixed quota hypothetical |
| Supplied UI/interface catalogue and editor reconstructed sources | Existing view/dialog mechanisms and editor confirmation/commit boundaries inspected | No new main-menu Co-op entry, lobby, generic intercept framework or source runtime equivalence established |

Private diagnostic identifiers and full log files are intentionally not published. Names of original mod diagnostic functions identify the measured code path without redistributing proprietary game source.

## Own native transport and diagnostics — 083/084, reviewed 2026-10-24

[Per-step ledger](DIAGNOSTICS_083_084.md) · [Measured architecture and limits](../networking/NATIVE_BRIDGE.md). Private logs were reviewed; no game test was launched for this update.

| Private evidence anchor | Finding | Boundary |
| --- | --- | --- |
| 083 interval-labelled PRE/POST GROUP/MEASURE and RESULT | Four successful 220-probe runs, 224 tasks joined each, fast post-pause samples after one-second tails at 50/250 ms | Supports history dependence; exact internal quota/scope unknown |
| 084 Step 6 Direct / Step 7 Return logs | Direct-string copies and game-managed returned strings, version/stack/result checks | Exact profiled game build, not generic Lua ABI |
| Step 8 native/game summaries | 4,100 rounds, 13 confirmed full GCs, 6,662 retained checks, both completions | Finite GC/endurance test |
| Step 9/10 native, game and Python records | Same-sandbox then cross-boundary pipe; external Python reports no SbieDll; 1,010 messages and three notices each | Local IPC, not WAN |
| Own LuaLog With/WithoutACSEDebug files | 1,000 fixtures checked line-by-line in order, zero content errors, 1,014 queued records each, workerError=0 | Mode labels are not installation detection; own diagnostic output only |
| Step 11 role A/B native logs and Python summary | 1,000 confirmed messages each way; wire counters 6,679/6,668 match Python; all completion markers; 112 reverse ACKs during held A message | Own logger used; no scenery effect, reconnect or production-network proof |
| Step 12 supplied code/instructions | Manual native-pipe Create and immutable duplicate test prepared | No reviewed outcome yet |
| Owner exit-crash report | Crash reporter appears on game termination, reportedly also without ACSEDebug | Root cause and fix unverified |

The own logger's completion footer records Lua's finalization request and native worker completion. Its post-close return acknowledgement is a separate observation. Private executable/code hashes, session identifiers, raw files and machine-specific paths are intentionally not published.
