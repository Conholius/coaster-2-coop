# Open questions and next experiments

**Updated 2026-10-23 through diagnostic 082.** [Status](STATUS.md) · [Latest ledger](DIAGNOSTICS_069_082.md) · [Latency](../networking/HTTP_LATENCY_080_082.md)

## Immediate: native HTTP delay

1. Does spacing requests at controlled intervals change the approximately 940 ms native wait? Account for control requests and prior request history.
2. Does a no-request recovery pause reset the behavior? Is scope per process, player/account context, endpoint or some other service queue? These are hypotheses, not established limits.
3. Does the wait precede TCP handshake/HTTP send or follow reply receipt? Server timing starts after accept; Python is a different process and sequential control. Add contemporaneous controls and a restricted TCP trace if required.
4. Are adaptive polling, batching, long polling, connection reuse or a sidecar practical with tested Kepler lifetimes/timeouts? None is yet a validated fix.

Larger backlog and separate ports did not reliably resolve 082. A fast server handler alone does not identify the native cause. Do not label a Frontier policy or fixed one-request-per-second limit as proven.

## Manual replication and ownership

- Extend corrected 079–081's single ordinary confirmed host Create to manual Move/Delete.
- Repeated placements, previews/cancel, group/dependency edits, rotation and scale beyond the specific diagnostic checks.
- Guest-originating edits, authoritative acceptance, concurrent conflicts, stale revisions and per-user Undo ownership.
- General origin/remote echo suppression: the 079–081 guest has capture/publication disabled; this is not a generalized bidirectional capture framework.

## Remote transport and product workflow

- Native remote HTTPS/certificate/auth/header/upload behavior, friend testing and packaging.
- The intended product direction is host opens a session, guest joins, both enter compatible world state and host controls saving. Independent local park loading is a diagnostic scaffold, not a final UX commitment.
- UI/interface archives suggest existing view/dialog/menu mechanisms worth investigating; an actual new Co-op main-menu entry and lobby are not runtime verified.
- Actual park/baseline transfer or negotiated compatible baseline; UI access alone does not provide it.
- Startup without cached contexts, Steam/Frontier outages and complete save/Cloud isolation remain separate tests.

## Recovery and storage

- Integrate a small custom registry through ACSE serialization callbacks; validate versions and missing/corrupt data.
- Bind logical mappings, revisions/tombstones and command status to the matching park checkpoint.
- Crash between native commit, status persistence, ACK and park save; no blind replay on unknown outcomes.
- Lost ACK reply after durable storage is distinct from refused/dropped ACKs before storage.
- Automatic mismatch reconciliation, late join, checkpoint compaction, retention and repair.
- Optional cross-session action history/counteractions and general Create/Delete data. Native history is unavailable after the tested reload.

## Performance coverage

- 069 split catalogue materialization from Lua search; 070 demonstrated targeted lookup. Broader resource/database behavior remains open.
- 068/069 do not retrospectively explain every earlier 067 outlier.
- Presented frames, profiling overhead, allocation/GC attribution, long-duration tests and realistic diverse parks.
- Do not extrapolate synthetic 100,000-entry registry tests to 100,000 real objects.
- Bound queues and native work while retaining completion checks; lower Advance count alone does not establish lower desync risk.

## Answered within the tested scope

Two simultaneous local clients; scripted Create/Move/Delete/native Undo/Redo and duplicate suppression; controlled two-client ACK/TCP/server restart recovery; completed-checkpoint Guest restart with Host alive; ordinary host Create while next preview stays local; targeted catalogue queries; completed 1,200-probe native/Python/server comparison.

These successes do not establish distributed exactly-once, arbitrary crash recovery, a public internet mod or full simulation synchronization.
