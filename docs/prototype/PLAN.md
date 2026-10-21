# First prototype: proof of persistent scenery replication

> **Status update — 2026-10-21, through diagnostic 034:** Scripted scenery create/move/delete, local save/reload, independent objects and session-local guarded replay have passed. Separate A/B runs matched canonical states after twelve inputs. **Transport, simultaneous clients and real-time replication remain untested.** See [runtime ledger](../research/DIAGNOSTICS_007_034.md) and [transport investigation](../networking/TRANSPORT.md).

## Goal

With two isolated PC2 clients on matching exact builds and independently loaded matching park baselines, reproduce a **confirmed** placement of one ordinary decoration prop on the second client and verify the result persists after a save/reload. This is a **plan**, not a completed test.

## Required milestones

Local portions have passed for the tested rock; cross-client portions below remain acceptance criteria. Next establish harmless message/ack transport, then integrate replay. Capturing ordinary confirmed user edits is still a separate requirement.

1. Identify retail game build, store, OS and executable SHA-256 for both clients.
2. Isolate copies of installs/test saves and verify authorized mod-loader operation.
3. Observe normal editor preview vs final commit. Trace function and arguments.
4. Record a canonical operation schema with explicitly typed IDs and world transforms. Do not use ephemeral local engine IDs as network IDs.
5. Replay a validated placement on a second instance without replacing whole-world state.
6. Save/reload both worlds and verify equality of the target prop, including persistence and absence of duplicates.
7. Extend to move, delete and undo/redo; document conflicts and rollback behavior.
8. Extend the transport/replay prototype with authority, ordering, versioning and recovery, then test packet loss, duplicates, disconnects and late joining.

## Test evidence

For each test log: exact build/hashes, operation sequence, save copy IDs (private), actual logs/screenshots (private until reviewed), expected vs observed outcome, reproducibility count, failures and changes to conclusions. Do not describe automated testing until automated clients were actually exercised.

## Initial exclusions

No guest/staff/economy simulation synchronization; no terrain or track editing, no ride synchronization, no public-online access, no anti-cheat bypass.
