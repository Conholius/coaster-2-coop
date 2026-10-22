# Current research status

**Updated 2026-10-22; reviewed runtime evidence through diagnostic 068. No playable co-op or simultaneous two-client synchronization yet.**

[Overview](../../README.md) · [Runtime tests](../testing/RUNTIME_TESTS.md) · [Latest ledger](DIAGNOSTICS_050_068.md) · [Measurements](../networking/BENCHMARKS.md) · [Recovery](../networking/RECOVERY_AND_HISTORY.md)

## Verified in a running game

Reported setup: Windows/Steam, PC2 **1.11.4.1011357**, ACSE **0.729**, ForgeUtils **2.7.0**, ACSEDebug **0.014**. Public executable/package hashes are not established. These are owner-run scripted diagnostics, not an automated simultaneous-client suite.

- **007–030:** AQ_Rock_04 scenery hierarchy/create/edit/delete/preview/cancel/commit, local save/reload, independent equal-resource objects and guarded canonical replay in separate A/B runs.
- **035b–047:** Lua/JS bridge; native Kepler GET/text transport, complete bodies/task joins, ready context without active Frontier connection; sequential local relay exchange, not simultaneous clients. HTTP loopback downloads up to 1 MiB passed.
- **048–054:** shared completion-token batches, up to 500 rocks per group and 2,701 rocks / 13 commits. Performance timer and Advance monitoring distinguish work from waits. New-group verification reduced checks; final inventory passed.
- **055–058:** HTTP DELETE, native live-session Undo/Redo for Create/Move/Delete, fresh entity mappings; tested duplicate/order/ID-conflict rejection without extra native changes.
- **059–062:** refused ACKs, real TCP resets and one journal-backed server restart recovered without duplicate native actions while the client ledger remained alive. All tasks joined in successful runs.
- **063:** controlled client restart after a completed saved checkpoint: mappings/command records restored, two old commands skipped, only a new Move committed. Not arbitrary crash recovery.
- **064:** saving preserved native Undo/Redo availability in the live session; after the tested restart both were unavailable, while object position/identity restored correctly.
- **065:** manually transferred saved Move data supported Undo/Redo effects through two new version-checked Move counteractions; duplicates/order/ID/version conflicts rejected. No native Undo/Redo calls in Phase 2; no automatic mod persistence yet.
- **066:** 12 synthetic registry/history-size combinations, 24 minimal/full variants passed through 100,000 records of each kind. Minimal lookups/updates remained cheap; allocations/building had occasional spikes.
- **067/068:** 2,701 real registered rocks, 72 Move groups / 1,464 object moves, 84 commits and 180,370 world checks passed. Final positions/versions matched. Native query bundles remained inexpensive in this setup; repeated full scans cost more than targeted checks.
- **068 profiling:** largest WORK callback 3.4665 ms, largest WORK Advance gap 21.6808 ms, none above 33 ms. A 37.2776 ms STARTUP callback was localized to catalogue result retrieval/search; it was not an ongoing edit cost.

## Derived design direction

Use a small mod-owned logical identity/revision registry, fresh native lookups, bounded ordered changes and application acknowledgements. Keep history bounded by a verified checkpoint policy. Prefer checking changed objects and distributing occasional full audits. HTTP remains a credible transport candidate; the local tests do not establish an internet edit-rate budget or full simulation synchronization.

Native Undo after a reload is not a safe recovery primitive. Command status is required for replay safety; optional cross-session Undo can use separately persisted action data and revision-checked counteractions. ACSE world-serialization callbacks are a source-supported persistence candidate, not yet a passed custom-registry integration.

## Not demonstrated

- Two simultaneous independently authorized clients, remote HTTPS, actual replicated scenery convergence or playable co-op.
- Capture/acceptance of ordinary player-confirmed edits and suppression of origin/remote echo.
- Automatic durable registry/history integration, arbitrary commit-before-recording client crash recovery, mismatched-save repair, late join or resync.
- Distributed exactly-once effects or atomicity across native commit, relay journal and park save.
- Broad resource, rotation/scale, groups/dependencies, terrain/path/track/ride/economy/AI support or multiplayer Undo ownership.
- Actual presented-frame measurement, long soak tests, 100,000 real scenery objects or a universal capacity limit.
- Steam-independent/cache-free startup, permanent service-outage operation, complete Cloud/save isolation or a validated second Sandboxie guest.

## Next milestones

Separate catalogue result materialization from its Lua search before optimizing that one-time startup path. For multiplayer, prove simultaneous harmless messaging, then a scripted scenery event with explicit authority, mappings and origin suppression. Test automatic serializer-backed checkpoint storage and ambiguous crash windows separately. The [HTTP design](../networking/HTTP_SYNC_DESIGN.md) remains a proposal connecting locally tested components.

005/006 retain their copied-install **outside Sandboxie** qualification. Later logs do not independently certify all process/Cloud boundaries. `UnstrippedToken=y` reduces token isolation; ACSEDebug GUI/native helper compatibility remains unvalidated.
