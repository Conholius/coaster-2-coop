# Open questions and next experiments

**Updated 2026-10-22 through diagnostic 068.** [Status](STATUS.md) · [Latest ledger](DIAGNOSTICS_050_068.md) · [Recovery](../networking/RECOVERY_AND_HISTORY.md)

## Multiplayer acceptance gates

1. **Simultaneous clients:** harmless numbered messages between two independently authorized real clients at the same time. 038's sequential exchange is not this proof.
2. **Remote route:** owner-controlled HTTPS relay, certificates/authentication/headers/timeouts and accessible host/join setup. A remote package was prepared, not validated by a friend test.
3. **Replicated scenery:** matching baselines/logical IDs, one accepted scripted event applied once per intended peer with matching resource/transform and origin echo suppression.
4. **Ordinary edit capture:** final player confirmation, preview filtering, ownership/acceptance and origin reconciliation.
5. **Concurrent edits:** authoritative ordering, stale revisions, permissions and per-action Undo ownership across actual peers.

## Recovery and persistent storage

- Test a small custom registry through ACSE world-serialization callbacks, including load timing, schema/version mismatch and missing data; source presence is not runtime persistence proof.
- Bind command status, revisions/tombstones and action data to the correct saved park checkpoint; restore Entity-IDs freshly.
- Crash between commit attempt, verification, durable status and saving; unresolved-outcome handling without blind replay.
- ACK reply lost after durable storage (distinct from 059–061's first ACK refusal/drop before storage).
- Automatic controlled client restart, mismatched/older saves, truncated/corrupt journals, generation/epoch changes, late join and repair.
- Safe real checkpoint compaction and tombstone retention; 066's pruning model is not durable cleanup proof.
- General Create/Delete counteractions and full transform/property/dependency data. 065 only tested a positional Move with manually transferred checkpoint text.

## Performance and transport

- Split 068's roughly 37 ms one-time catalogue result retrieval/search into narrower measurements; evaluate a validated lookup/cache rather than claiming a proven cache optimization.
- Reproduce the earlier 067 transition outliers: 068 did not reproduce them in WORK and cannot identify their earlier cause retrospectively.
- Presented frame-time capture, profiling overhead, allocation/GC attribution, camera/park-content controls, more resources and long-duration tests.
- No linear extrapolation from 066's 100,000 synthetic records to 100,000 real objects. 067/068 stop at 2,701 simple rocks.
- Native POST/body/headers, remote TLS, redirects, compression, connection reuse, long polling, uploads and multiple concurrent clients remain untested.
- Request cancellation/world-owner transitions, bounded queues, backpressure and native task lifetimes under prolonged failure.
- Fresh uncached/offline profiles, Steam outage and permanent Frontier shutdown; ready Kepler context without active Frontier connection is the tested boundary.
- Exact package/build provenance and complete Steam/Cloud/guest isolation.

## Answered within the tested scope

Local scenery create/move/delete persistence; native loopback HTTP and text; relaxed Kepler readiness; shared-token grouped creation to 500; live-session native Undo/Redo; tested HTTP duplicate/order/ID-conflict guards; live-client recovery after ACK refusal/TCP reset/server restart; controlled saved-checkpoint client reload; absence of native history after the tested reload; positional Move counteractions; synthetic minimal/full registry measurements and real-object targeted/full checks. See the [050–068 ledger](DIAGNOSTICS_050_068.md) and [earlier transport ledger](DIAGNOSTICS_035_049.md).

No distributed exactly-once, automatic general recovery or playable co-op claim follows.
