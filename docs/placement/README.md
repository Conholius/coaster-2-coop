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

This section records the earlier **static scenery-specific editor call path**. Later runtime diagnostics below established the tested create/move/delete and local save/reload path; remote replay and cross-client identity remain unverified. Do not execute decompiled Lua with unresolved `PC2VEC...` string placeholders. `DraggableWidgets` handles interactive transforms, whereas `TranslationGizmo` creates temporary visual prefabs.

## Minimum proof required

1. Capture a regular scenery action after mouse release and final confirmation.
2. Identify the underlying durable creation/edit/delete call and argument structure.
3. Replay a harmless simple prop placement in a disposable copy of a park.
4. Save and reload; verify the object, transform and placement inventory remain correct.
5. Repeat with move/delete; test undo and redo semantics.

**Updated:** scripted create/move/delete and local persistence are now demonstrated for AQ_Rock_04 (009–025). Capturing ordinary player-confirmed edits remains unverified. Native live-session Undo/Redo for the tested Create/Move/Delete later passed in 056–061; cross-client history ownership remains open.

## ACSE / ForgeUtils follow-up (2026-10-20)

[Source-pinned investigation](ACSE_FORGEUTILS.md) distinguishes catalogue registration from placed instances and identifies a PC2 track mod's concrete preview/commit lifecycle. That investigation predates the runtime creation and persistence evidence below.

## Runtime update — editor presence confirmed (2026-10-21)

Diagnostic **006** in the copied PC2 test installation **outside Sandboxie** successfully read `worldAPIs.editors` in active `tropicalworld` and found `CreateUndoOperationsHierarchy` plus `api.undo.PreviewChangeList`, `CanCommitPreview`, `CommitPreview` and `CancelPreview` as functions. The minimal world component with a `Shutdown()` handler also survived the menu-to-park transition.

**Historical boundary:** 006 itself was presence-only. The later diagnostics supersede its planned-007 status.

## Runtime update — creation, movement, deletion and persistence

Diagnostics 007–025 successfully exercised hierarchy/change-list construction, AQ_Rock_04 creation, StartEditingPart_Scenery, transform changes, completed preview/cancel/commit, local save and verify-only manual reload. The test part 1182 persisted at (10,5,10), later at (15,5,10), and was absent after saved deletion. 026–030 added independent equal-resource objects and guarded ordered command replay.

See the [full lifecycle, exact tested signatures and limitations](../research/DIAGNOSTICS_007_034.md). Part IDs in that record belong to one test park; entity IDs can change after cancellation/reload. Cross-client IDs must be mod-owned and mapped locally. Native loopback transport was subsequently verified in 037b–047, but normal player-edit capture and global engine determinism remain unproven.


## Grouped creation update — diagnostics 048/049 (2026-10-22)

Both created 274 unique rocks with 16 group previews/commits/checkpoints and 32 world checks, without diagnostic saves. Issuing all CreateNewPart requests with one shared completion token before waiting reduced tested up-to-50 groups to one preparation Advance; preview used two and commit one. This is successful grouped request submission/completion, not proof of concurrent CPU execution or universal group support. See [measurement boundaries](../networking/BENCHMARKS.md). Later 050–054 verified 2,701 rocks / 13 commits with groups to 500. 067/068 add real-object register/query/Move measurements; see the [050–068 ledger](../research/DIAGNOSTICS_050_068.md).

## Undo, mapping and query update — 055–068

HTTP DELETE and native live-session `Undo()` / `Redo()` passed with completion-token waits and fresh entity mapping checks. 056 retained the captured Part-ID while Entity-IDs changed on native recreation. 064 retained saved object state but lost native-history availability after the tested reload; 065 instead applied version-checked Move counteractions from manually transferred action data. [Recovery/history boundaries](../networking/RECOVERY_AND_HISTORY.md).

067/068 registered 2,701 actual rocks and verified 1,464 scripted moves, 84 commits and 180,370 world checks. Minimal registry records contain logical key/local Part-ID/revision/deletion state; snapshots used as expected values are diagnostic data. The tested native existence/resource/entity/transform queries remain inexpensive; repeated full scans cost proportionally more. [Timings and limitations](../networking/BENCHMARKS.md).
