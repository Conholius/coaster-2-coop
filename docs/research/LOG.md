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
