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

## Minimum proof required

1. Capture a regular scenery action after mouse release and final confirmation.
2. Identify the underlying durable creation/edit/delete call and argument structure.
3. Replay a harmless simple prop placement in a disposable copy of a park.
4. Save and reload; verify the object, transform and placement inventory remain correct.
5. Repeat with move/delete; test undo and redo semantics.

**Not yet demonstrated.**

## ACSE / ForgeUtils follow-up (2026-10-20)

[Source-pinned investigation](ACSE_FORGEUTILS.md) distinguishes catalogue registration from placed instances and identifies a PC2 track mod's concrete preview/commit lifecycle. The scenery-specific creation arguments and save/reload proof remain unresolved.
