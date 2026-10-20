# Evidence index

These are names of locally researched materials, **not bundled source files**. The original copies remain under ignored `local/` or a separately maintained authorized evidence archive. The dates/builds below are file-label claims, not necessarily the version of the installed executable.

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
| Bytecode inspection | original `.lua.bin`, converted headers and `unluac` disassembly | PC2 Lua 5.3 custom format; conversion works for many files | Some decompilations fail despite valid disassembly |

## Selected anchors in reconstructed files

- `managers.parkloadsavemanager_unluac.lua`: `_RequestSaveParkToFile`, `GenerateCurrentParkMetaData`, `DoPendingParkLoad`.
- `managers.localchangemanager_unluac.lua`: `_CacheSaveIfLocalChange`, `_FindLocalChanges`, `DeleteOutdatedLocalChanges`.
- `api.debug_unluac.lua`: `API_GetListOfLoadedModuleNames`, `API_ReloadModule`.
- `common.moduleutils_unluac.lua`: `DisallowIndexNewIndexExcept` and metatable guards.

**Next evidence improvement:** attach stable source build identification, reproducible extraction steps and hashes of original artifacts in private research records (not proprietary contents). 

## Public source follow-up (2026-10-20)

See [ACSE / ForgeUtils investigation](../placement/ACSE_FORGEUTILS.md) for commit-pinned source references and function-level anchors. ForgeUtils catalogue registration, its Lua hooks, ACSE's tracked-entity serializer and PC-CC's track preview/commit implementation were inspected. These are static source observations; none establishes PC2 scenery save/reload persistence.
