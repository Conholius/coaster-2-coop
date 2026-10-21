# Research status (2026-10-20)

## Verified in inspected static material

- PC2-provided Lua/native API names include editor undo hierarchy and placement mapping functions.
- `Game.GameScript` configures `Game.GameEnvironment`, and `BaseGame` sets up APIs and world transitions.
- `api.debug.GetListOfLoadedModuleNames()` wraps `package.loaded`; `API_ReloadModule` uses `loadresource`, `tryrequire` and `pcall`. Debug shell command registration is conditional on `api.debug.IsDebugAllowed()`.
- `ParkLoadSaveManager` calls `worldserialisation:RequestSaveWorldToFile`, polls `HasSaveToFileCompleted`, and coordinates park loading and world transitions.
- `LocalChangeManager` caches save metadata for online parks and tracks save/autosave tokens and version numbers.
- `CommonObjectCreator` calls `api.entity.InstantiatePrefab` with prefab, transform, parent and per-instance properties; one example uses a `TransformPrefab` as an entity parent.
- Scenery and part data `.specdef` material contains placement-related identifiers, resource names and transforms.

## Newly traced in PC2 scenery editor Lua (static evidence)

- `SubModePathSnapPlacement:BeginSubMode` creates an undo hierarchy, creates/edits scenery parts, and starts a preview.
- `_CreateMoveObject_NewInstance` uses `CreateNewPart`, waits on a completion token, then calls `StartEditingPart_Scenery`. `_CreateMoveObject` also branches for moving and cloning.
- `_DoPlace` validates the preview and marks `bCommitChanges`; `ExitSubMode` later calls `CommitPreview` or `CancelPreview`. `_DoDelete` marks a deletion within the same edit session.
- `scenery_volumeextras:CalculateBestPlacementPosition` computes snapping and the placement transform. `DraggableWidgets` manages gizmo movement, rotation and scale; it is not the commit boundary.
- Four editor `.lua.bin` files contained 19 nonstandard vector constants. The recovered Lua contains `PC2VEC...` placeholders, unsuitable for execution before value restoration and validation.
- No mod-triggered placement, save/reload persistence, event capture or multiplayer synchronization has been tested.

## Inferred, not yet verified

- Confirmed editor changes could be recorded as operations and replayed through an editor / undo interface; which calls occur and whether they are externally invokable is unknown.
- Whole-world saves could form the initial shared park baseline, with a separate network layer carrying changes after initial load; this is an architectural hypothesis.

## Not yet demonstrated

- Persistent scenery creation/modification/deletion through a custom Lua call.
- Capturing user-confirmed placement events, stable remote identifiers, or conflict resolution.
- Two-PC or two-client synchronous co-op, multiplayer networking, full save/load parity, or playability.

## Important caveat

The study references PC2 dumps bearing distinct build identifiers, including `1.0.6.429191` and a prefab dump labelled `1.10.2.906765` (2026-06-25). **Version equivalence must not be assumed.** Record exact build and SHA-256 of the game's main executable before conducting runtime tests.

## Runtime validation and debugging warning (2026-10-21)

**Verified in a running isolated PC2 installation (Windows; game's self-reported build 1.11.4.1011357; ACSE 0.729; ForgeUtils 2.7.0; ACSEDebug 0.014):**

- A custom Cobra `PC2CoopDiagnostics` `Main.ovl` was loaded and its `LuaDB.Init()` ran; `api.debug.Trace()` emitted `[PC2-COOP]` lines in `Planet Coaster 2.log`.
- `require("forgeutils.hookmanager")` succeeded and returned a table with `AddHook`; ForgeUtils logged version 2.7.0 and successful initialization.
- With a custom ACSE `AddLuaComponents` world component enabled, the `startscreenglobe` world reported that `api.world.GetWorldAPIs()`, `worldAPIs.editors.CreateUndoOperationsHierarchy`, and `api.undo.PreviewChangeList/CanCommitPreview/CommitPreview/CancelPreview` existed. **These were presence checks in the menu world only; no method call, valid undo hierarchy, or park persistence was demonstrated.**
- With that custom world component registered, creating a tropical sandbox park caused a game crash during the `startscreenglobe -> tropicalworld` transition, after `ACSEComponentManager:Shutdown()`. The log had no definitive exception/stack trace.
- **Control test:** Commenting out `LuaDB.AddLuaComponents` and rebuilding the mod, while **leaving ACSEDebug 0.014 active**, allowed a tropical sandbox park to load and run for over 60 seconds, with scenery editing UI invoked and a later normal shutdown. This strongly implicates the custom component or its interaction with world transitions, not ACSEDebug alone; exact cause unknown.

**Cross-game ACSEDebug caveat:** This debug build produced useful logs under PC2, but was not established as fully PC2-compatible. The debug menu was **not opened in the successful control test**; user-reported or community-reported menu crashes are not reproduced as part of this logged control. The provenance of this particular ACSEDebug binary/package (Planet Zoo vs PC1) is not independently established in these logs. Avoid assuming native functionality or menu safety.

**Next engineering step:** establish a minimal ACSE world-component lifecycle that safely survives world deactivation/shutdown and recreation in the park world; then repeat presence-only editor API checks **inside tropicalworld**. Never perform object edits until this is stable.
