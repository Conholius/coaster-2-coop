# Persistent scenery placement: ACSE / ForgeUtils investigation

Checked: 2026-10-20. **Static source research only; no PC2 runtime or save/reload test was performed.**

## Result

The strongest candidate for ordinary editable park scenery is the game's editor undo/change-list pathway, entered from an ACSE mod and optionally instrumented through ForgeUtils hooks. ForgeUtils's scenery builder registers a catalogue definition; it does not place an instance. ACSE also contains a separate entity serialization mechanism, but its compatibility with the installed PC2 package/build has not been established.

## Source-pinned observations

1. [ForgeUtils SceneryPartBuilder](https://github.com/Distantz/ForgeUtils/blob/90b78c31693a473e73d621af145053c3b5fc8059/Main/forgeutils/builders/scenerypartbuilder.lua)
   - `asSimplePart()` selects the `SurfaceScaling` data prefab.
   - `withID()` sets the scenery resource identity; `withVisualPrefab()` selects visuals.
   - `addToDB()` writes ModularSceneryParts, control points, simulation, UI data, tags and optional scaling database rows.
   - There is no world position, world instance creation, undo commit or save operation in this builder. Registration alone therefore cannot establish persistence of a placed instance.
2. [ForgeUtils database initialization](https://github.com/Distantz/ForgeUtils/blob/90b78c31693a473e73d621af145053c3b5fc8059/Main/database/forgeutilsluadatabase.lua)
   - `Init()` hooks `Managers.BrowserDataManager._SetupCache`.
   - It invokes `Database.Main.CallOnContent("InsertToDBs")` before the original cache setup.
   - For a new custom catalogue part, registration belongs in the mod's `InsertToDBs` lifecycle. Required prefab/localization/tag resources must also exist. The same mod content must be available when loading the park.
3. [ForgeUtils HookManager](https://github.com/Distantz/ForgeUtils/blob/90b78c31693a473e73d621af145053c3b5fc8059/Main/forgeutils/hookmanager.lua)
   - `AddHook(moduleName, functionName, hookMethod)` passes the next/original function as the first callback argument.
   - Hooks target functions on required Lua modules. This is not evidence that native userdata methods such as hierarchy operations can be hooked directly.
   - A hook must preserve original arguments, return values and error behavior. Its successful registration before module load is not proof that it later executed.
4. [PC-CC importer manager](https://github.com/parker02311/PC-CC/blob/e722a387d407e532d0c6be914a97b7a0121f21b7/Mod_Importer/Main/Managers/Mod_Importer.UIManager.lua#L251)
   - This PC2 mod provides a concrete example of the general preview/commit sequence.
   - `Init` obtains the world's `editors` interface.
   - `BeginPreview` creates an undo operations hierarchy, adds operations, obtains its full change list, calls `api.undo.PreviewChangeList`, and waits for the returned operation token.
   - `TryCommit` checks `api.undo.CanCommitPreview`, calls `CommitPreview`, then waits again.
   - Waiting polls `api.undo.IsOperationComplete(token)` from a coroutine resumed by the manager.
   - **Boundary:** the actual inserted operation is `api.track.AddSelection`. This is a track example, not a demonstrated scenery operation or a save/reload test.
5. [ACSE component manager](https://github.com/OpenNaja/ACSE/blob/8a843e61bcb57779778536e635d334d882cee7b1/Source/ACSE/Main/components.standalonesceneryserialisation.lua#L405)
   - Registers a `StandaloneScenerySerialisation` world serialization client.
   - Tracks entities supplied to its component manager with a non-nil `PrefabName`.
   - Saves prefab name, transform, mapped entity ID and mapped parent ID.
   - Loads by mapping saved IDs and calling `InstantiatePrefab`, with a completion token.
   - Arbitrary spawned entities are not automatically included: membership in the tracked component set matters.
   - **Boundary:** this source includes explicit Planet Zoo behavior. Its presence in upstream ACSE does not prove this implementation is shipped or usable in the PC2 ACSE package.
6. [ACSE PC2 distribution](https://www.nexusmods.com/planetcoaster2/mods/1)
   - The author lists a PC2 release 0.723 and a last verified game version 1.0.7.434846. This identifies a PC2 distribution; it does not certify the user's current build or prove the upstream serializer matches that package.

## Candidate sequence for a native scenery instance

This is a research sequence, not runnable code with verified scenery arguments:

1. Load an isolated test park and obtain its world `editors` interface.
2. Create an undo operations hierarchy.
3. Add the **exact scenery creation operation and argument structure captured from a normal confirmed placement**. Existing research identifies `CreateNewPart(s)` as candidates, but their signatures and required part data are still unknown.
4. Get the full undo change list.
5. Call `PreviewChangeList`; wait for the operation token to complete.
6. Check `CanCommitPreview`; inspect failure reasons on rejection.
7. Commit the preview; wait for completion before attempting a save.
8. Verify a real placement identity and object selection/editing behavior.
9. Save, leave the park, reload, and compare resource identity, count and transform.

Steps 2 and 4–7 now have a concrete PC2 mod source reference. Step 3 is the unresolved scenery-specific bridge. A completed commit is still not proof of save persistence.

## Recommended first experiment

Use an existing base-game freeform decoration, avoiding custom catalogue assets for the first persistence proof.

- Record exact PC2 executable build/hash, ACSE package version/hash and ForgeUtils version.
- Capture one manually confirmed placement through a narrow Lua editor hook. Log argument types/values needed for the operation, resource name, transforms and returned IDs; do not serialize an entire game module.
- Identify the Lua caller of `CreateNewPart(s)` and the correct part data construction. Reproduce its ordering, coordinate frame and completion handling.
- Replay once into a separate empty test area. Do not run the spawn routine automatically on every park load.
- Compare counts before and after saving/reloading. Repeat a second reload with replay disabled to rule out automatic respawn and duplicates.
- Move the prop, save/reload again; delete it, save/reload again. Check undo/redo separately.
- Treat changed runtime EntityIDs as expected possibilities; compare durable identity/resource/transform rather than equating runtime IDs across sessions.

## Current blockers and next inputs

The current workspace has no game/API dumps or isolated PC2 installation. The exact scenery creation call and argument layout cannot yet be extracted from the previously referenced local materials. Required next inputs are the location of those materials and an authorized isolated test environment. No game installation, save, or mod package was modified in this investigation.
