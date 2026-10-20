# Park save/load and component serialization

## ParkLoadSaveManager

`ParkLoadSaveManager` orchestrates park saves; the active world's `worldserialisation` interface performs the requested serialization. Observed methods include `RequestSaveWorldToFile`, `HasSaveToFileCompleted`, `RequestSaveTerrainToFile`, `RequestSaveBlueprintToFile`, `LoadWorldFromFile`, `LoadTerrainFromFile` and `LoadBlueprintFromFile` (availability and invocation paths vary).

A save request is initiated and completion polled asynchronously using Lua coroutines. Park loads may be staged as a request, followed by `api.game.RequestTransition`, then serviced by `DoPendingParkLoad` in the target world. The specific world serialization implementation and underlying file format are still unknown.

## LocalChangeManager is not an editor edit log

`LocalChangeManager` reads `api.save.GetSaveMetadata`, keeps local save/autosave tokens under online park IDs and version numbers, and can delete outdated local versions. No evidence in the reviewed module of per-scenery mutation events or a real-time replication mechanism.

## Component specs

- `standalonesceneryserialisation.specdef`: `PrefabName`, transform child structure.
- `voxelterrainserialisation.specdef`: terrain prefab/material references, transform and voxelterrain child structure.
- `voxelterrainunderlayserialisation.specdef`: voxel terrain underlay structure.
- `guestserialisation.specdef`: guestbrain structure.
- `staffserialisation.specdef`: prefab, staffdata and staffsimulation structures.
- `worldserialisationtest*.specdef`: small test component definitions, including a test with primitive string/u32/u64/float fields.

Component schemas **do not establish** the complete save schema, how objects receive persistent IDs, or how loaded park parts are reconstructed. Never infer the format of a live network message directly from these partial component declarations.
