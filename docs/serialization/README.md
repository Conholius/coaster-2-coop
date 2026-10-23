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

## Runtime persistence — diagnostics 019, 022 and 025

The own Lua diagnostic created AQ_Rock_04, saved a new local park and found the same test part 1182/resource at (10,5,10) after manual reload. A later move to (15,5,10) persisted in 022. Deletion and absent entity mappings persisted in 025. Each verification used a separate verify-only code phase, without recreating or reapplying the action.

The tested manager path obtains Interfaces.IParkLoadSaveManager through the game environment, calls GenerateCurrentParkMetaData and SaveParkToSaveToken with the game owner plus a new local save descriptor, waits for completion and api.save.IsBusy() to clear, and checks the returned token/name. An undo Checkpoint is separate from a park save.

[Diagnostic ledger and limits](../research/DIAGNOSTICS_007_034.md). This proves the tested scenery effects survive these local saves, not the full serializer format or persistent mod-owned network-ID metadata. Full Steam Cloud isolation, cross-client IDs and general persistence remain open.


## Distributed recovery remains open (2026-10-22)

Later native HTTP and grouped editor tests do not extend the save proof to network identity metadata. A park commit, relay journal and applied-command watermark are not an atomic transaction. Persist logical object keys/revisions/tombstones with an agreed checkpoint and verify reconciliation before claiming crash-safe replay or late join. [Proposed recovery rules](../networking/HTTP_SYNC_DESIGN.md).

## Controlled restart and history findings — 063–065

063 paired two completed commands with one verified new local save and a server-held checkpoint manifest. After client restart, Phase 2 verified save identity and current world, refreshed entity mapping, restored those records, skipped two replays and committed only a new Move. The new Move was not saved. This is completed-checkpoint recovery, not arbitrary crash-safe serialization of the mod.

064 observed native Undo/Redo available immediately before/after saving, then both unavailable after the tested restart. 065 kept explicit Move before/after data and manually transferred a checkpoint string; new revision-checked counteractions restored Undo/Redo effects without invoking native history. Automatic persistent action/command state is still open.

The public ACSE component manager registers world-serialization callbacks and maps save/entity references; a matching ACSE load callback appears in PC2 logs. This is a concrete candidate for co-saving custom state, not a completed integration or proof that `SaveEntityID` is a peer identity. [Source anchors, recovery boundaries and next tests](../networking/RECOVERY_AND_HISTORY.md).

## Two-client completed-checkpoint extension — 078

Guest B saved one new local checkpoint after Create/Move, restarted while Host A and the relay remained alive, validated the matching save/checkpoint, refreshed native Entity-ID mapping and restored two applied command records. Replayed old effects were skipped; only the new Move to (20,5,10) committed after reload. B saved once in Phase 1 and did not save again in Phase 2. [Per-phase counts and limits](../research/DIAGNOSTICS_069_082.md).

This is controlled checkpoint recovery, not automatic ACSE custom-registry serialization, arbitrary crash safety or a final guest-save policy. The intended product may use host-controlled saving and a lobby; those UI/world-transfer/metadata workflows are still unverified.
