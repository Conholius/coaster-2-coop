# First prototype: HTTP-delivered scenery replication

> **Updated 2026-10-22 through diagnostic 049.** Local native HTTP text transport, sequential A/B messaging, HTTP-triggered local CREATE/MOVE, guarded replay and up-to-50 grouped creation are verified. **Simultaneous two-client replication, normal edit capture and distributed persistence are not.** Diagnostic 050 has no reviewed result.

[Current facts](../research/STATUS.md) · [Proposed protocol](../networking/HTTP_SYNC_DESIGN.md) · [Native contract](../networking/TRANSPORT.md)

## First acceptance goal

With two independently authorized real PC2 clients on matching builds and park baselines, reproduce one scripted host-accepted AQ_Rock_04 event on the guest. Compare logical identity/resource/world transform and intended commit counts; local Part-IDs may differ. First run without saves. This is a plan, not a completed test.

## Steps

1. Record exact game/mod versions and hashes privately; confirm copied-install/profile/Cloud boundaries and matching park baseline/identity manifest.
2. Exchange harmless numbered messages concurrently over a local route, then the intended owner-controlled remote route; retain full acknowledgements and task lifecycle evidence.
3. Establish session/epoch/version validation, authoritative ordering, immutable command-ID records, per-client logical-ID mapping and origin echo suppression.
4. Apply one host-accepted scripted CREATE to the guest; wait for native completion and verify resource/position/mapping across two Advances before application ack.
5. Test redelivery, response loss, order gaps, changed-content ID conflicts and stale revisions without extra effects. Stop unresolved commit outcomes rather than retrying blindly.
6. Extend to MOVE/DELETE and independently addressed identical resources; validate error/cancel/undo ownership.
7. Add ordinary confirmed user-edit capture as its own tested feature.
8. Design durable journal/checkpoint/object metadata integration; only then test explicit save/reload, crash/reconnect, resync and late join. Local save tests alone are insufficient.
9. Use measured batch/frame budgets and stress/soak tests; keep full simulation outside the first scope.

## Evidence required

Exact build, operation/session sequence, canonical states, command and commit counts, task completions, failures and explicit expected/observed outcomes. Keep raw logs, park files and personal paths private. Distinguish owner-run scripted diagnostics from an automated two-client suite. Protocol recovery rules in the design page are proposals until implemented and exercised.

## Exclusions

No guest/staff/economy simulation, terrain/paths/tracks/rides, public service mutation, anti-cheat bypass or claim of playable co-op. Do not publish game dumps or tokens.
