# Recovery, object identity and undo history

**Evidence through diagnostic 068, reviewed 2026-10-22.** [Ledger](../research/DIAGNOSTICS_050_068.md) · [Proposed protocol](HTTP_SYNC_DESIGN.md) · [Serialization](../serialization/README.md)

## Three different kinds of state

| State | Meaning | Tested example |
| --- | --- | --- |
| Command processing status | Not started / in flight / applied and verified; immutable command identity and payload | Retained DONE records suppress retries after ACK loss and resets in 059–062 |
| Action data and action state | Resource/transform data needed for a counteraction; APPLIED or UNDONE | 065 positional Move stores before/after and changes action state without native Undo |
| Park checkpoint | Which object state actually exists in a particular saved park | 063 binds completed command sequence and mapping manifest to a verified new local save |

An Undo command can be DONE while the original action is UNDONE. A relay ACK is not proof an effect is included in the saved park. A deleted object may still need a logical tombstone and history data.

## What the recovery tests establish

- **059:** first ACK request receives 503 without being stored; redelivery finds DONE and does not repeat the native action.
- **060/061:** TCP reset without HTTP response before ACK storage; the live client retains its in-memory ledger and resumes without duplicate Create/Move/Delete or Undo/Redo.
- **062:** Python server process exits and reloads an atomically replaced journal; generation/boot checks distinguish the restarted server. The still-running client's DONE record handles one replayed Move.
- **063:** controlled client restart after one verified new local save and a sealed checkpoint. Phase 2 checks save identity/resource/position, refreshes entity mapping and then restores two checkpoint-included command records. Two replayed commands are skipped; only a new Move commits.

063 verifies old command **payloads** against checkpoint records while verifying the **current** world against the checkpoint's latest state. The original Create position is not expected to be the current position after a checkpointed Move. The later unsaved Move is runtime state, not an update to the saved checkpoint.

These are single-client local tests. They do not establish simultaneous peer recovery, distributed exactly-once effects, arbitrary client-crash safety, late join or automatic repair of a mismatched save.

## Remaining ambiguous window

There is still no demonstrated atomic transaction across native editor commit, client/relay durable status and park save. A crash after commit but before durable recording can leave an uncertain outcome. Reserve the command before native work, stop parallel retries, and reconcile an explicit checkpoint/inventory after restart. Never assume that a network failure means the object was not changed.

A timeout must not release a still-running native request/task and start an overlapping retry. Detect owner/world changes and preserve unknown-outcome state until reconciliation. Those transition/cancellation/soak cases still need tests.

## Native history versus counteractions

056/057 establish native Undo/Redo for the tested live-session Create, Move and Delete. No-argument `Undo()` / `Redo()` return completion tokens. Recreated entities can have different Entity-IDs; re-read mappings after each operation.

064 found both native history directions available immediately after saving, but neither available after the tested client restart and reload. The object state persisted. Therefore a prototype can begin a new native history after loading, while still restoring the command ledger needed for safe replay.

065 demonstrates an optional cross-session **Move counteraction**:

1. Record logical object key, original action ID, before/after position and object revision, paired with a saved checkpoint.
2. After restart, verify the saved world and resolve fresh mappings before trusting that record.
3. Apply Undo as a new Move to the old position, with its own command ID and expected revision.
4. Apply Redo as another new Move to the recorded new position.
5. Skip identical redelivery; reject changed-content IDs, wrong sequence and stale revision unchanged.

This is not engine-stack reconstruction. 065's checkpoint was copied manually from a log into the next-phase source. Automatic persistent mod storage is not yet implemented. Create/Delete counteractions would require a separate test; complex objects may need properties, parent/group relationships and other dependencies beyond resource and position.

Native history from different clients must not be assumed identical. Shared Undo over a network needs explicit authority and operation ownership. Cross-session Undo is optional for the first scenery prototype; stable identity and reliable command processing are core requirements.

## Minimal registry proposal

Keep only logical key → local Part-ID, revision and deletion/tombstone state in the live registry. Read Entity-IDs and object properties through the game's placement APIs when needed. Retain canonical payload/status/outcome records for the agreed replay window, and bounded action data only where the supported history feature needs it. Avoid continuously mirroring the entire park or scanning all objects on each edit.

066 validates synthetic minimal/full records up to 100,000 objects and 100,000 history entries; 067/068 validate a minimal registry with 2,701 real rocks and 1,464 scripted moves. This is not a universal capacity limit. See [measurement boundaries](BENCHMARKS.md).

Pruning requires a real durable checkpoint and agreed retention policy. 066's quarter-history deletion is a synthetic model only. Older commands below its watermark are rejected as BELOW_CHECKPOINT, not claimed to have passed exact payload comparison after their records were removed. Do not discard tombstones or replay evidence before all consumers/recovery paths can safely advance.

## Candidate automatic persistence: source observation only

The public ACSE `PlanetCoaster2` branch and `main` resolved to commit `8a843e61bcb57779778536e635d334d882cee7b1` during inspection. Its [component manager](https://github.com/OpenNaja/ACSE/blob/8a843e61bcb57779778536e635d334d882cee7b1/Source/ACSE/Main/components.standalonesceneryserialisation.lua#L130) calls `RegisterWorldSerialisationClient` with save/load/version callbacks and unregisters them at shutdown. Its tracked-entity serializer uses [SaveEntityID / LoadEntityID](https://github.com/OpenNaja/ACSE/blob/8a843e61bcb57779778536e635d334d882cee7b1/Source/ACSE/Main/components.standalonesceneryserialisation.lua#L405). These mapped save references are not cross-client network IDs.

The installed PC2 logs contain `WorldSerialisationClient_ACSE_Load`, corroborating an ACSE load callback in the tested package. It does not prove that our component's arbitrary tables can be saved/reloaded safely through that interface, or that the installed implementation is byte-identical to public source. A small dedicated save/load test, versioning and callback-lifecycle validation are still required.

[ForgeUtils ModDB](https://github.com/Distantz/ForgeUtils/blob/90b78c31693a473e73d621af145053c3b5fc8059/Main/forgeutils/moddb.lua) registers mod/version requirements; it is not a placed-object or action database. Native part sets are useful selection/group containers, not a demonstrated durable synchronization ledger. No inspected API removes the need to define our own peer identity, revisions and command semantics.
