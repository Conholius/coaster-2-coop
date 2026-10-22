# Research log

## 2026-10-20 — Consolidation

- Organized prior static research on PC2 Lua environments, editor registries, undo API names, entity spawning, save/load orchestration and component schema material.
- Distinction reaffirmed: an entity spawned using `InstantiatePrefab` is not known to be a persisted park placement.
- `LocalChangeManager` disambiguated: manages local versions of online park saves, not per-object collaborative editing events.
- Key next step identified: find confirmed scenery edit commit call site and test save/reload persistence on an isolated park.
- No successful real-time synchronization, multiplayer code or live in-game validation yet.

Future entries: record date, test build + SHA-256, original symbol/source, test procedure, observed behavior, failure mode and remaining uncertainty.

## 2026-10-20 — ACSE / ForgeUtils placement investigation

- Inspected pinned upstream sources: ForgeUtils catalogue builder and hook lifecycle, ACSE entity serialization, and PC-CC's PC2 undo preview/commit sequence.
- Confirmed in source that SceneryPartBuilder writes catalogue database rows rather than creating park instances.
- Found a concrete PC2 preview/wait/check/commit/wait example; its operations are track-specific, so scenery creation remains unproven.
- Documented the exact missing bridge and reload/duplicate-control experiment in [the placement investigation](../placement/ACSE_FORGEUTILS.md).
- No runtime tests performed; local game dumps and an isolated test environment are not available in the current workspace.

## 2026-10-20 — Scenery editor call-path breakthrough (static evidence)

- Investigated locally supplied `editors.scenery.rar` and four original `editors.scenery.*.lua.bin` files: `translationgizmo`, `axispositioning`, `draggablewidgets`, `submodepathsnapplacement`; corresponding decompilations were inspected without committing game source.
- The four bytecode sources held 19 custom PC2 vector constants, temporarily replaced with `PC2VEC...` strings during decompilation. These are placeholders, not runtime-valid vector objects. Decompiled code remains subject to bytecode cross-checks.
- `SubModePathSnapPlacement:BeginSubMode` constructs a change-list hierarchy, creates an editable scenery move object, positions it and calls `StartPreview`. `_CreateMoveObject` distinguishes create, clone and move; `_CreateMoveObject_NewInstance` calls `CreateNewPart`, waits on a completion token and calls `StartEditingPart_Scenery`.
- `_DoPlace` checks placement validity and preview committability, then sets `bCommitChanges = true`; `ExitSubMode` is the observed deferred `CommitPreview` / `CancelPreview` boundary. The delete path destroys the edit-session move object and sets the commit flag.
- `scenery_volumeextras:CalculateBestPlacementPosition` returns snapping flags and `placementTransform`; `DraggableWidgets` maintains interactive transform/scale state. A plain `InstantiatePrefab` call in `TranslationGizmo` constructs temporary editor visuals rather than a demonstrated permanent park part.
- Public docs updated to reflect this stronger **static** scenery evidence; do not infer runtime ACSE/ForgeUtils compatibility, persistent park state or network synchronization.
- **Next test:** use a disposable park, log a confirmed simple scenery edit, try a controlled mod-initiated create/commit using verified runtime APIs, save/reload twice with replay disabled, then test move/delete and undo/redo.

## 2026-10-21 — PC2 ACSEDebug logging and world-component A/B test (runtime evidence)

- Test context: isolated Windows PC2 installation, **PC2 1.11.4.1011357** (reported by ACSE at log startup), **ACSE 0.729**, **ACSEDebug 0.014** (cross-game debug package), **ForgeUtils 2.7.0**, custom `PC2CoopDiagnostics` Cobra mod; package-specific SHA-256 and executable SHA-256 not yet recorded. Logs are kept private and **not committed**.
- Baseline debug log: `[PC2-COOP] Diagnostics started`, `ACSE version: 0.729`, `ForgeUtils require success: true`, `HookManager type: table`, `AddHook available: true`, `Diagnostics completed`. ACSEDebug wrote `Planet Coaster 2.log` and the game reached the main menu.
- **Component enabled (failure):** `LuaDB.AddLuaComponents` registered `PC2CoopDiagnosticComponent`; `Configure` and `Init` executed. On menu-world activation it logged `GetWorldAPIs succeeded: true`, editor API presence and undo API presence. New tropical sandbox park creation proceeded through menu-world deactivation and `ACSEComponentManager:Shutdown()`, then crashed before the subsequent park-world initialization appears in the log. No stack trace or definitive cause.
- **Component disabled (control):** Removed/commented out only the custom `LuaDB.AddLuaComponents` registration, rebuilt `Main.ovl`; retained **ACSEDebug 0.014** and the existing `LuaDB.Init()` ForgeUtils check. New tropical sandbox park **loaded successfully** (`World Fade-in completed`), remained stable for **over 60 seconds**, entered foliage/scenery placement UI, then exited without the preceding crash.
- **Interpretation (inference):** custom world component's lifecycle / cross-world teardown / integration is a strong crash candidate; **missing `Shutdown()` was not experimentally isolated** and must not be called the root cause. ACSEDebug can provide log output in this setup despite cross-game provenance; **do not infer debug-menu safety** or unrestricted PC2 compatibility. Debug UI not activated in successful control run.
- **Next A/B test:** register a stripped-down component with only supported lifecycle callbacks, confirm deactivate/shutdown ordering and reinitialization on park transition; add editor API presence checks back one at a time only after a successful park load.

## 2026-10-21 — Minimal world component, packaging diagnosis and park-world APIs

- The first attempted minimal-component runs (`Planet Coaster 2(3).log`, `(4).log`) loaded successfully but **did not register** `PC2CoopDiagnosticComponent`; the intended rebuilt `Main.ovl` had not actually been loaded. These are not component-on success tests.
- Diagnostic **005** added a `VERSION TEST 005` marker. `Planet Coaster 2(5).log` showed the correct rebuilt `Main.ovl`, `AddLuaComponents` registration, menu-world `Configure`, `Init`, and `OnWorldActivation`.
- The continued test (`Planet Coaster 2(6).log`) showed old-world `OnWorldDeactivation` and `Shutdown`, then recreation and `OnWorldActivation` of the minimal custom component in `tropicalworld`. Park fully loaded, remained running, screen mode was changed from in-game settings, and the world shut down normally.
- This passing minimal version **both** added `Shutdown()` **and** removed previous editor probes. **Neither change was individually isolated as the cause of fixing the earlier crash**.
- Diagnostic **006** (`Planet Coaster 2(7).log`) reintroduced **field and method presence checks**. In `tropicalworld`, `GetWorldAPIs` succeeded, `worldAPIs.editors` and `api.undo` were accessible, and the named `CreateUndoOperationsHierarchy` and `api.undo` methods were recognized as functions. This run loaded and deactivated its park normally.
- **Critical testing caveat:** The tester subsequently confirmed that these game runs used the **copied test installation but not Sandboxie**. Earlier descriptions of an "isolated installation" must not be mistaken for verified Sandboxie process isolation.
- A separate Sandboxie host/guest setup and previous `UnstrippedToken=y` launch compatibility are described in [the translated notes](../ISOLATED-INSTALLATION.md). The user-reported main-menu start in that configuration does not establish that diagnostic 006 was run inside Sandboxie, nor prove the failure was due to Denuvo.
- **Next planned diagnostic 007 (not yet run):** Invoke `editors:CreateUndoOperationsHierarchy()` in a disposable park; report return type and available hierarchy methods; no creation, preview or commit of scenery objects.
- Repository documentation reorganized around [current status](STATUS.md), [runtime test matrix](../testing/RUNTIME_TESTS.md), [topic index](../README.md) and [active blockers](OPEN_QUESTIONS.md). Existing historical failure logs remain in this chronology; no raw logs or binaries were published.

## 2026-10-21 — Diagnostics 007–034: durable scenery and transport boundary

- Consolidated owner-run tests and earlier private log reviews: hierarchy invocation, change list, AQ_Rock_04 creation, editor transform access, cross-Advance checks, preview/cancel and single commits succeeded.
- Creation persisted at (10,5,10) in 019; existing-part movement persisted at (15,5,10) in 022; deletion persisted in 025. Mutation/save and manual reload verification were separate code phases, preventing accidental replay during loading.
- 026/027 proved independent handling of two identical resources using mod keys and local part IDs.
- 028: eight inputs, four duplicate skips, four commits. 029: four order rejections, four conflicting-ID rejections, four valid commits.
- 030: twelve inputs in separate A/B-labelled runs; all 120 canonical STATE records matched after label/timestamp removal. Repeatability only, not actual client synchronization.
- 031 found no standard io/os/LuaSocket access; planned file roundtrip skipped. package.loadlib present but not invoked.
- 032 raw enumeration was incomplete. 033 found five named fields as functions. 034 resolved those same five controls, with all nine invented names nil, no errors and Completed=true. Neither read-only test invoked the resolved functions.
- Static workshop/newsfeed/account code uses specialized request handles and structured result getters. Arbitrary URLs and raw HTTP response text remain unknown; no CreateKeplerHttpRequest call site found in supplied material.
- Fully decompiled the supplied FrontierAccountManager using unluac --nodebug after debug-informed decompilation failed at PrepareToShutdown. Parsed all 135 bytecode prototypes, produced 134 function declarations plus main and a full disassembly. Original files unchanged; analysis outputs stay private; reconstruction not executed.
- Rechecked retained 022/025/030/034 evidence for this documentation update. Earlier test results were not rerun. Full ledger: [007–034](DIAGNOSTICS_007_034.md); next blocker: [transport](../networking/TRANSPORT.md).
- Updated current docs to supersede planned-007 and no-persistence statements, retaining historical failures and outside-Sandboxie 005/006 qualifications. No second simultaneous client, confirmed player-edit capture, distributed recovery or playable co-op is claimed.


## 2026-10-22 — Native HTTP, offline context and batched scenery (035–049)

- Consolidated prior owner-run 035b–049 log/control reviews, reread retained 038 A/B/A2 logs and 047 native worker/server evidence; no game run during documentation work.
- Lua-JS roundtrip passed; JS external HTTP did not reach loopback server. Independent Python in the same sandbox passed; private resource-handler investigation supports using native HTTP instead.
- Generic Kepler contract established: constructor playerID, SetUrl/GET/SetGenerateAuthCode(false), Spawn worker, RunKeplerRequest third response with nResponseCode/sBody, JoinAsync lifetime handling. Early 037 cancellation is historical; later successful tests supersede "unknown transport".
- 038 exchanged messages through a retained server between sequential runs; A-again completed receipt and repeat sends were duplicates. This is not simultaneous co-op. Remote friend test prepared but not completed.
- 039 validated 16 inputs; 040–042 applied local HTTP CREATE/MOVE and suppressed repeated commands without extra commits.
- 043 local requests passed after Frontier loss, but recovery incomplete. 044 early no-context request failed, later offline-start requests passed after context appeared. 045 confirms readiness does not require active Frontier connection. No Steam/cache/service-independent startup claim.
- Correct 046 online/offline and settled/mixed 047 downloads passed. 047: 110/110, full content, joins and releases; no strong size bottleneck to 1 MiB locally, not internet bandwidth proof.
- 048/049 each passed 274 unique rocks, 16 commits/checkpoints and 32 world checks. Shared completion reduced 50-object preparation from 50 Advances to one; pipeline from 53 to four. Same-frame clock under-resolution and trace overhead documented.
- 050 code supplied: larger groups/increasing population, planned 2,701 objects / 13 commits. Owner testing; outcome unreviewed, no success claim.
- Added native transport contract, evidence ledger, bounded performance interpretation and a proposed host-authoritative HTTP relay protocol with logical identity, application acks, backpressure and unresolved-commit/crash boundaries. Updated current status/unknowns/next objective; historical failures retained.
- Public scope: documentation and sanitized findings only. No raw logs, game binaries/resources, OVLs, saves, private endpoints/tokens or personal Windows paths.

## 2026-10-22 — Review through 068: scale, recovery, history and attribution

- Reread retained version-marked owner logs 050–068. No game run during documentation work; earlier failed 057/060/068 runs remain failures, corrected repeats supply completion evidence.
- 050/051: 2,701 rocks / 13 commits; minimal own logging did not materially change coarse game-clock timing. 052 introduced a performance timer and Advance monitoring. 053 reduced verification to new groups; 054 added a passing final inventory, 8,103 checks total.
- 055–058: HTTP DELETE and local/native HTTP-driven Undo/Redo passed, with rejected duplicates/order/ID conflicts preserving state and refreshed entities.
- 059–061: ACK refusal then real TCP resets before first ACK storage resumed without duplicate native actions. Correctly interpret native failure tuple rather than treating status field 2 as an HTTP response. Successful 061 joined all 38 tasks.
- 062: actual journal-backed server process restart with surviving client ledger. 063: controlled client restart after one saved/sealed checkpoint; old commands skipped and only new Move committed. Arbitrary commit/status/save crash windows remain unresolved.
- 064: native history available after saving but unavailable after tested reload. 065: manually transferred Move data enabled two new counteractions and version/ID/order/duplicate guards, not native-stack persistence.
- 066: synthetic minimal/full registry/history comparison passed 12 combinations / 24 layouts. Real durable pruning and exact retained memory are not established.
- 067: real 2,701-object registry, 1,464 moves / 72 groups, 84 commits and 180,370 checks passed; targeted checks stayed much cheaper than full inventories. Earlier MIXED_TRANSITION outliers lacked attribution.
- Corrected 068: same workload; 6,867 callbacks paired, section accounting passed. One startup catalogue result-handling/search section took 37.2313 ms; WORK own/gap maxima 3.4665/21.6808 ms, no WORK gaps over 33 ms. Earlier WORK outliers did not recur; no retrospective causal proof. Instrumentation overhead explicitly retained.
- Reviewed native-registry and ACSE/ForgeUtils alternatives. ACSE world-serialization callbacks are a source-supported automatic checkpoint candidate, not a verified custom-registry integration. ForgeUtils ModDB is a version registry. Minimal own identity/revision/status semantics remain necessary.
- Updated overview, current status, runtime/evidence/unknowns, performance, recovery/history, placement, serialization and proposed HTTP integration. Preserved repaired Mermaid syntax. No raw logs, proprietary scripts/assets, OVLs, saves, checkpoint tokens or personal paths published.
