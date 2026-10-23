# Current research status

**Updated 2026-10-23; reviewed through diagnostic 082. Two local clients and manual host Create replication passed. No released playable co-op or full simulation synchronization.**

[Overview](../../README.md) · [Runtime tests](../testing/RUNTIME_TESTS.md) · [Latest ledger](DIAGNOSTICS_069_082.md) · [HTTP latency](../networking/HTTP_LATENCY_080_082.md) · [Design](../networking/HTTP_SYNC_DESIGN.md)

## Verified in a running game

Reported environment: Windows/Steam, PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0, ACSEDebug 0.014. Public executable/package hashes are not established. Tests are owner-run diagnostics, not an automated multiplayer QA suite.

- **007–034:** local hierarchy/Create/Move/Delete, explicit commits, save/reload, independent identical resources and guarded separate-run replay.
- **035–047:** native loopback GET/text and relaxed Kepler readiness. Tested requests work without active Frontier connectivity once a valid Kepler user context exists. Original 038 was sequential; its later tropical-park repeat passed simultaneous numbered exchange.
- **048–054:** batching, shared completion tokens, 2,701 rocks / 13 commits, groups up to 500, performance timing and targeted/final audits.
- **055–065:** local native Undo/Redo, validation, controlled ACK/reset/server and completed-checkpoint client recovery; native history absent after the tested reload. Positional Move counteractions from manually transferred data worked.
- **066–068:** synthetic registries to 100,000 entries; 2,701 real registered rocks, 1,464 moves, 84 commits and 180,370 checks. Synthetic entry counts are not physical-object capacity.
- **069:** full 7,263-row GetAllResults 32.5976 ms, first Lua search 4.2579 ms. Twenty retained-table searches averaged 3.9730 ms.
- **070:** 110 targeted known/unknown queries passed with no full catalogue fetch/search; maximum own callback 0.1188 ms and paired Advance gap 18.5428 ms.
- **071–074:** two simultaneous local clients each applied scripted Create/Move/Delete and native Undo/Redo, with matching tested world state and unchanged duplicate effects.
- **075–077:** both clients survived refused first ACKs, real TCP aborts and a controlled journal-backed server restart without repeating applied native actions.
- **078:** Guest B saved one new checkpoint, restarted while Host/server remained alive, restored records/mappings, skipped old effects and applied only a new Move. This is a controlled completed-checkpoint workflow.
- **Corrected 079:** an actual ordinary host editor commit was captured and replicated to B once; its duplicate was skipped.
- **080/081:** that confirmed placement replicates while the distinct next preview on A remains local and active in all sampled observations. Neither test sends an unconfirmed preview.
- **082:** all 14 phases and 1,200 probes passed: A 480, B 600, sandboxed Python 120; no bad bodies, all native tasks joined and all participants/server completed. No editor or save calls.

## Current latency finding

**Observed:** 080 took 591.284 ms from original host commit return to confirmation on A; 081 took 2,239.214 ms. These include proof/poll/verification gates. In 081, the B commit call was 0.017 ms, completion wait 16.231 ms, and command reception to completed commit 136.577 ms.

**Observed:** 082 starts with fast single-client probes; during the shared phase requests develop approximately one-second tails. Later B medians remain about 944–947 ms even alone, with a larger backlog and on a separate listener. The delay is inside RunKeplerRequest, not worker dispatch or Join. Server processing/write sections remain fast; Guest Python medians are 0.94–0.99 ms, max total 24.04 ms. [Full measurements and limits](../networking/HTTP_LATENCY_080_082.md).

**Hypothesis:** request-history-dependent pacing, limiting or another internal request-path wait. No fixed quota, limiting scope, Frontier policy, TCP retransmission cause or exact Kepler implementation has been demonstrated. Time before server accept and delayed completion after reply remain unresolved.

## Derived design direction

Use a minimal logical-key/local-Part-ID/revision registry, immutable command status, fresh Entity-ID lookups, ordered accepted events and application acknowledgements. Transmit changed supported objects rather than continuously retransmitting every unchanged object. Keep sampled verification and recovery outside the delivery-critical path where safely validated; retain native completion and duplicate guards.

HTTP remains a candidate. Kepler's observed request behavior must be measured separately from HTTP as a protocol. Adaptive polling, bounded batching, long polling, persistent connections and a local sidecar are options to test, not proven fixes. Fewer Advances are not by themselves an optimization target: distinguish CPU work, completion observation, verification and visual frames.

## Not demonstrated

- Remote HTTPS/friend testing, authentication, uploads, long polling or persistent-connection behavior.
- General ordinary manual Move/Delete capture, repeated/cancelled placements, arbitrary resources/full world transforms, groups and dependencies.
- Simultaneous conflicting edits, authoritative ownership and multiplayer Undo policy.
- Automatic in-save mod-registry persistence, arbitrary client-crash atomicity, mismatched-save repair, late join or resync.
- A production host/join lobby, complete automatic shared-save workflow or released playable co-op.
- Full guest/staff/economy/terrain/path/track/ride simulation, universal capacity or presented-frame guarantees.
- Steam/cache/service-shutdown independence or complete Cloud/save/account isolation.

## Next milestones

1. Vary request spacing/count and recovery pauses; distinguish pre-connect delay from post-response native completion, using a TCP trace if needed.
2. Extend actual manual host capture to Move/Delete, then repeated placement, cancellation and Undo ownership.
3. Validate a remote secured relay and practical host/join UX separately from world replication.
4. Integrate durable metadata/checkpoints and explicit ambiguous-crash reconciliation before promising general recovery.

Historical 005/006 outside-Sandboxie findings remain unchanged. Later successful Host/Guest tests supersede the old "guest unvalidated" status only for the tested arrangement; they do not certify every isolation boundary.
