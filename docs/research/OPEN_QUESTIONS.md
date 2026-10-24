# Open questions and next experiments

**Updated 2026-10-24 through diagnostic 084 Step 11; Step 12 pending.** [Status](STATUS.md) · [Latest ledger](DIAGNOSTICS_069_082.md) · [Latency](../networking/HTTP_LATENCY_080_082.md)

## Immediate: own native transport and diagnostic lifecycle

1. Execute 084 Step 12: manual Host Create to Guest via the DLL/companion path, next preview kept local, one Guest commit and immutable duplicate skip. It is prepared, not yet verified.
2. Investigate game-exit crash reports with no-mod / ACSE-only / own-DLL controls and final lifecycle evidence. Do not attribute the cause to ACSEDebug or the worker without isolation.
3. Test pipe disconnection, companion restart, slow/unavailable peers, bounded queues, cancellation and world deactivation. Historical HTTP recovery tests do not certify the new channel.
4. Provide a reliable bootstrap/fallback when the DLL cannot load; the own logger does not capture every engine message or native crash.

083 answered the spacing/quiet-pause experiment within a finite local scope. History-dependent pacing is supported, but a fixed quota, its scope and exact implementation remain unknown. Kepler is set aside as the preferred path; the local DLL/pipe alternative has passed through Step 11. [Findings](../networking/NATIVE_BRIDGE.md).

## Manual replication and ownership

- Extend corrected 079–081's single ordinary confirmed host Create to manual Move/Delete.
- Repeated placements, previews/cancel, group/dependency edits, rotation and scale beyond the specific diagnostic checks.
- Guest-originating edits, authoritative acceptance, concurrent conflicts, stale revisions and per-user Undo ownership.
- General origin/remote echo suppression: the 079–081 guest has capture/publication disabled; this is not a generalized bidirectional capture framework.

## Remote transport and product workflow

- Implement and test the external companion's remote authenticated transport, encryption, friend testing and packaging. Local Named Pipes do not provide a WAN link; HTTP or another remote protocol remains an independent choice.
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

Own profiled direct string exchange under GC, own logging without ACSEDebug and bidirectional local DLL/pipe messaging with 1,000 messages each way also passed. These successes do not establish distributed exactly-once, arbitrary crash recovery, a public internet mod or full simulation synchronization.
