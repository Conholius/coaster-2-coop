# Current research status

**Updated 2026-10-22, runtime evidence through diagnostic 049. Diagnostic 050: prepared / owner testing, no reviewed result.**

[Overview](../../README.md) · [Runtime record](../testing/RUNTIME_TESTS.md) · [Transport](../networking/TRANSPORT.md) · [Proposed synchronization](../networking/HTTP_SYNC_DESIGN.md)

## Verified in a running game

Reported setup: Steam/Windows, PC2 **1.11.4.1011357**, ACSE **0.729**, ForgeUtils **2.7.0**, ACSEDebug **0.014**. Exact binary hashes are not established publicly.

- 007–025: scripted AQ_Rock_04 create/edit/move/delete, preview/cancel/commit; local create/move/delete persistence after new local save and separate manual verify-only reload. Earlier test part 1182 is a local park ID, not a network identity.
- 026–030: independently addressed identical rocks, guarded ordering/duplicate/ID-conflict checks; twelve-input canonical replay matched in separate A/B runs. 030 had no networking.
- 035b: Lua/JavaScript roundtrip and UI resource cleanup. Gameface external HTTP attempts did not reach the local server; native HTTP replaced this route.
- 037b–047: CreateKeplerHttpRequest(playerID), SetUrl/GET/SetGenerateAuthCode(false), RunKeplerRequest in an api.task.Spawn worker; third return carries nResponseCode and sBody. Tasks are retained until JoinAsync completion.
- 038: actual messages stored and retrieved through a local relay in sequential A, B and A-again runs, three each; repeated A messages recognized as server duplicates. No simultaneous client pair.
- 039: HTTP-delivered 16-input validation: four accepted, four duplicates, four order errors and four ID conflicts, no game changes.
- 040–042: HTTP CREATE and MOVE applied once to the local test park; duplicate redelivery caused no additional commit. DELETE over HTTP has not yet been connected/tested; local deletion is established separately.
- 043–045: local HTTP continued after Frontier disconnect and worked on the tested offline launch once Kepler context was ready. A valid game owner, IsKeplerAvailable and IsKeplerUser are the tested readiness gate; active Frontier IsConnected is not required. No success before Kepler user context, no cache-free/Steam-independent bootstrap proof.
- 046/047: loopback downloads through 1 MiB passed full body checks and task completion; 047 passed 110/110 requests after settling/mixed order. No strong payload-size bottleneck in this limited local test.
- 048/049: 274 unique rocks, 16 grouped commits/checkpoints, 32 world checks. Shared-token creation reduced 50-rock preparation from 50 Advances to one; pipeline from 53 to four. Up to 50 per group verified; no diagnostic saves.

Details: [007–034](DIAGNOSTICS_007_034.md), [035–049](DIAGNOSTICS_035_049.md), [measurement boundaries](../networking/BENCHMARKS.md).

## Derived assessment

HTTP is a credible candidate for ordered scenery-change delivery. Transmit bounded change batches and occasional inventory/checkpoint data, not a full park every frame. Current data do not establish remote latency, safe maximum edit rate, uploads, concurrent clients or full park simulation synchronization. 048's linear waits were substantially our sequencing strategy, not a proven hard engine limit. No transport replacement is justified solely by these local measurements.

## Not yet demonstrated

- Two simultaneous independently authorized game clients; internet/remote HTTPS test; playable co-op.
- Capture of ordinary player-confirmed scenery edits and suppression of origin/remote echo.
- Durable mod-owned identity mapping, save/baseline manifests and deletion tombstones across peers/reloads.
- Host/relay authority, stale-revision handling, application acknowledgements, crash-safe recovery, reconnect, late join or resync.
- Atomicity between a park commit and a durable command record; distributed exactly-once effects.
- General scenery/rotation/scale/group support, multiplayer undo/redo, terrain/paths/tracks/rides/economy or AI simulation.
- Large-park performance: 050's planned 2,701-rock test has no reviewed outcome yet.

## Context and next milestones

005/006 were explicitly copied-install runs outside Sandboxie; retain that qualification. Later controls established Python loopback from PC2CoopHost and owner-run native loopback, not a fully validated second guest setup or complete Steam/Cloud isolation. UnstrippedToken=y reduces token isolation; it does not identify Denuvo as the failure cause. ACSEDebug logging works, cross-game GUI/native helpers remain unvalidated.

Next: review 050 separately, then test two simultaneous clients with harmless messages, remote routing and a scripted scenery event. Keep normal edit capture and persistent recovery as explicit subsequent milestones. The [HTTP protocol design](../networking/HTTP_SYNC_DESIGN.md) is proposed, not shipped code.
