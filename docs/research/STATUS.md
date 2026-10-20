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
