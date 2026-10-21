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
