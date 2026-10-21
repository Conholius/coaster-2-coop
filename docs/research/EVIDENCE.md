# Evidence index

These are names of locally researched materials, **not bundled source files**. The original copies remain under ignored `local/` or a separately maintained authorized evidence archive. The dates/builds below are file-label claims, not necessarily the version of the installed executable. The first table and 005/006 entries preserve their historical evidence scope; current runtime results through 034 are indexed in the later section and in [STATUS.md](STATUS.md).

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

## Separate local Sandboxie setup (not the above runtime runs)

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
