# Scenery placement and undo — critical research area

## Relevant API symbols from the inspected PC2 API inventory

- `Game::Editors::UndoOperationsHierarchy`: `CreateNewPart`, `CreateNewParts`, `EditExistingPart`, `EditExistingParts`, `DestroyExistingPart`, `DestroyExistingParts`, `GetFullUndoChangeList`, `CreateCompositeObject` (symbol inventory; call signatures not comprehensively verified).
- `Casino::LuaScript::API::Undo`: `CreateChangeList`, `PreviewChangeList`, `CanCommitPreview`, `CommitPreview`.
- `Game::LuaScript::API::Placement`: `EntityIDToPlacementID`, `PlacementIDToEntityID`, `PlacementIDToAllEntityIDs`, `GetPartResourceName`, `PlacementPartExists`.

## Identity and coordinates

Keep `EntityID`, `PlacementID`, `PlacementPartID`, `SaveEntityID`, and any cross-network ID distinct. A prefabricated child's placement field defaulting to zero is not proof of a valid active-world placement ID. Do not interchange child-local and world transforms.

## Related components

- `partdata.specdef`: placement-part identifier and shared part structure.
- `partdataresourcename.specdef`: resource-name part data.
- `partdatatransform.specdef`: optional override transforms.
- `scenerypuppet.specdef`: `PlacementPartID` field.
- `standalonesceneryserialisation.specdef`: `PrefabName` and transform reference.

## Comparative mod observations

Enhanced Move Tool hooks scene draggable widgets and reads/manipulates transform presentation; the reviewed code did not demonstrate a durable `Undo.CommitPreview` call. ProTrack hooks track-edit preview methods and selected track entities; track behavior cannot be assumed to apply to scenery placement. Cross-title Frontier mods are conceptual references only.

## Scenery edit call path identified in decompiled code (2026-10-20)

The new local evidence is `Editors.Scenery.SubModePathSnapPlacement` (`pathsnap.lua`) and `editors.scenery.brushplacement_unluac.lua` from the scenery archive. In `SubModePathSnapPlacement`, `BeginSubMode` creates an undo hierarchy and starts a preview; `_CreateMoveObject_NewInstance` calls `CreateNewPart(resourceName, completionToken)`, waits for completion, then calls `StartEditingPart_Scenery`. The clone and move branches use `CloneExistingPart(s)` and `EditExistingPart(s)`. `_PositionObjects` calls `scenery_volumeextras:CalculateBestPlacementPosition`, and `_DoPlace` validates the preview before setting `bCommitChanges`. `ExitSubMode` is the deferred `CommitPreview` / `CancelPreview` boundary. `_DoDelete` destroys the edit-session move object and marks a commit.

This establishes a **static scenery-specific editor call path**; runtime mod access, successful save/reload, remote replay and object ID stability remain unverified. Do not execute decompiled Lua with unresolved `PC2VEC...` string placeholders. `DraggableWidgets` handles interactive transforms, whereas `TranslationGizmo` creates temporary visual prefabs.

## Minimum proof required

1. Capture a regular scenery action after mouse release and final confirmation.
2. Identify the underlying durable creation/edit/delete call and argument structure.
3. Replay a harmless simple prop placement in a disposable copy of a park.
4. Save and reload; verify the object, transform and placement inventory remain correct.
5. Repeat with move/delete; test undo and redo semantics.

**Not yet demonstrated.**

## ACSE / ForgeUtils follow-up (2026-10-20)

[Source-pinned investigation](ACSE_FORGEUTILS.md) distinguishes catalogue registration from placed instances and identifies a PC2 track mod's concrete preview/commit lifecycle. The scenery-specific creation arguments and save/reload proof remain unresolved.

## Runtime update — editor presence confirmed (2026-10-21)

Diagnostic **006** in the copied PC2 test installation **outside Sandboxie** successfully read `worldAPIs.editors` in active `tropicalworld` and found `CreateUndoOperationsHierarchy` plus `api.undo.PreviewChangeList`, `CanCommitPreview`, `CommitPreview` and `CancelPreview` as functions. The minimal world component with a `Shutdown()` handler also survived the menu-to-park transition.

**Boundary:** These are **method-presence checks**, not successful invocation, preview, commit or persistence. The **next planned** diagnostic 007 is to call `editors:CreateUndoOperationsHierarchy()` and inspect the returned hierarchy without adding edits. See [runtime experiment record](../testing/RUNTIME_TESTS.md#next-experiment-diagnostic-007).
