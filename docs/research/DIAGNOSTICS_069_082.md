# Diagnostics 069–082: two clients, manual placement and HTTP latency

**Reviewed 2026-10-23.** [Current status](STATUS.md) · [Latency analysis](../networking/HTTP_LATENCY_080_082.md) · [Protocol design](../networking/HTTP_SYNC_DESIGN.md)

## Evidence boundary

These are owner-run diagnostics, not a newly executed automated QA suite. Retained private diagnostic logs and original diagnostic server/component code were reread for this update; older intermediate reviews supply additional context. No game was launched during documentation work. Raw logs, saves, extracted scripts, account data and personal paths remain private.

The reported test environment is Windows/Steam, PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0 and ACSEDebug 0.014. The HTTP servers/control use Python 3.11.0 in the completed 082 run. This evidence establishes two local running game clients in the tested Host/Guest arrangement, not two remote machines, licensing portability or complete Steam/Cloud isolation.

Successful corrected runs supersede the earlier failed attempts below. Matching a local Part-ID across clients is incidental; the logical object key is the cross-client identity. Entity-IDs must be resolved locally and refreshed after recreation/load.

## Catalogue profiling: 069 and 070

**Verified in a running game — 069:** one result retrieval and a complete traversal of 7,263 rows located AQ_Rock_04. The first callback measured GetAllResults at **32.5976 ms** and Lua search at **4.2579 ms**; total measured work was 36.8575 ms, callback 36.8829 ms and following Advance gap 48.0136 ms. Twenty searches over the retained table averaged 3.9730 ms (3.8463–4.6738 ms). Completed=true; no create/edit/commit/save calls.

**Derived:** full result materialization dominated the measured catalogue section, with a smaller but nonzero Lua-search cost. This locates the earlier roughly 37 ms startup section more precisely; it does not explain all earlier unrelated transition spikes, prove a cold cache, or measure presented frames.

**Verified — 070:** 110 targeted queries passed: five statements × known/unknown resource × first pass plus ten warm repetitions. There were 55 known and 55 unknown queries, 110 targeted result fetches, zero full-catalogue queries and zero Lua catalogue searches. Statements: HasDatabaseEntry, GetPartDataPrefab, GetContentPackForPart, GetBrowserLabel and GetPuppetPrefabName, bound by SceneryPartName. Known resource: AQ_Rock_04; unknown resource handled according to each statement's count/empty-result semantics.

Targeted fetch maxima were at most **0.0176 ms** in the reported summaries; total query elapsed includes asynchronous waits, generally about 11–12 ms and one Advance per query. Maximum own callback was 0.1188 ms; maximum paired Advance gap 18.5428 ms, none above 33 ms. Completed=true, no world/save changes.

**Derived:** targeted lookup avoids the giant result retrieval and full Lua scan for this use case. GetPartDataPrefab's data-prefab identifier must not be confused with the visual puppet/resource name. Database collation and broader resource coverage are not established by one resource.

## Simultaneous message exchange

A later tropical-park-only repeat of 038 passed with a fresh shared session: both running clients sent and received three numbered messages and completed. The original sequential A/B/A-again 038 results remain historical; the later repeat supplies the simultaneous-client evidence. This is loopback on one machine, not an internet/friend test.

## Two-client scripted scenery and recovery

All successful runs below use matching disposable tropical test parks and the allowlisted AQ_Rock_04 resource. Per-client counts are not global counts.

| Diagnostic | Verified result | Boundary |
| --- | --- | --- |
| 071 | Each client created and committed one rock; identical second delivery skipped; A/B tasks 54/54 and 7/7; both completed | Server-originated scripted command; no ordinary edit capture |
| 072 | Each client created at (10,5,10), moved to (15,5,10), one Create commit and one Move commit, duplicate Move skipped; A/B tasks 24/24 and 10/10 | One supported object; server log was not retained for the reviewed run |
| 073 | Each client created and deleted one rock; Create/Delete commits one each, duplicate Delete skipped; absent part/entity mappings; A/B tasks 18/18 and 10/10 | No save |
| Corrected 074 | Nine unique Create/Move/Delete/Undo/Redo actions from 18 deliveries; nine duplicates skipped per client; three commits, three Undo and three Redo calls; fresh mappings, 46 checks; A/B tasks 66/66 and 48/48 | Native live-session history, not shared ownership or post-reload stack persistence |
| 075 | Nine deliberately refused first ACKs per client, 18 deliveries, nine unique effects and nine duplicates; eventual nine accepted ACKs; A/B tasks 54/54 and 43/43 | First ACK refusal before storage, not loss of a reply after durable storage |
| 076 | Nine real first-ACK TCP aborts per client, nine transport failures and nine duplicates, nine unique effects; A/B tasks 55/55 and 43/43 | Client ledger survives; server cleanup correction removes premature wfile.close |
| 077 | Server process restarted with resume, generation 2; each client resumed with nine unique actions, ten deliveries and one duplicate; four transport failures each; A/B tasks 98/98 and 70/70 | Controlled server restart, not arbitrary client-crash atomicity |
| 078 | Guest B saved one new local checkpoint after Create/Move, restarted while A/server stayed alive, restored two command records and mappings, skipped old effects and applied only new Move to (20,5,10) | Explicitly paired completed save/checkpoint; no automatic arbitrary-save repair |

All listed final successful client runs report Completed=true with joined tasks and no unintended extra native actions. Earlier 074 registration/session handling and 076 server-cleanup failures are not counted as successful final runs.

### 078 phase accounting

- A remained alive: Create once, Move twice, three commits, no saves, 173/173 tasks.
- B Phase 1: Create once, Move once, two commits, exactly one permitted new local save, 11/11 tasks. Phase1Completed=true and overall Completed=false are intentional at this handoff.
- B Phase 2: two restored command records, zero Create calls, one new Move/commit, three duplicate encounters across four deliveries, no further saves, 13/13 tasks. Final Completed=true.
- Entity mapping was refreshed after load; native Undo/Redo history loss remains consistent with 064. This does not restore a native history stack or close the commit-before-record/save crash window.

## Manual Host Create: corrected 079

**Verified in a running game:** the ForgeUtils hook on Editors.Scenery.SubModePlacement.CommitPreview was actually invoked, not merely registered. The original editor commit returned, one committed Part-ID was captured and one event published. Host diagnostic-owned create/edit/preview/commit/checkpoint counts stayed zero. Guest performed exactly one Create/commit/checkpoint, skipped an identical second delivery and completed.

Successful corrected run: A 7/7 tasks and four world checks; B 23/23 tasks and six world checks; both Completed=true. Source Part-ID was 1182; host/guest Entity-IDs differed. The localized browser label was confirmed as “Aquatischer Stein – Vorsprung 4” for AQ_Rock_04.

Earlier attempts included an initial unavailable-server request and a final assertion caused by a transient gap between old/new previews. The corrected 079 deliberately waited for the placement tool to exit. Therefore 079 alone did not prove immediate replication while the next preview stayed active.

**Source observation:** the inspected editor's scenery-placed notification precedes commit and is not sufficient evidence of a durable accepted placement. The derived hook queues observation data after the original method, preserving its arguments, yielding, errors and return values; it does not run HTTP inside the hook. Public hook-manager anchor: [ForgeUtils AddHook implementation](https://github.com/Distantz/ForgeUtils/blob/90b78c31693a473e73d621af145053c3b5fc8059/Main/forgeutils/hookmanager.lua). Runtime invocation is established by the diagnostic logs, not this source link alone.

## Active next preview: 080

**Verified:** one manually confirmed host stone was replicated while a different next preview remained active. Host committed Part-ID 1182 and preview Part-ID 1183 were distinct. All **45/45** observations found the same next preview active; interruptions=0. The unconfirmed preview was not transmitted.

Guest: Create=1, commit=1, checkpoint=1, deliveries=2, duplicate=1, six world checks. Host: one capture/original editor commit/publication, zero diagnostic native mutation calls, four checks. A/B HTTP tasks **53/53 and 69/69**, all 122 responses HTTP 200, both Completed=true, save calls zero.

Host-clock intervals from original commit return: next preview confirmed **194.339 ms**, publication response/task completed **263.152 ms**, guest application confirmation returned to A **591.284 ms**. The last interval is not the guest commit duration or pure HTTP latency. It includes preview readiness, checks, an additional A proof gate, polling, native work and the confirmation path. Preview continuity is sampled; not every presented frame was observed.

## Separated timing: 081

**Verified:** both completed; A/B tasks **93/93 and 124/124**; Guest Create/commit/checkpoint once, duplicate skipped; A preview active in **348/348** observations, zero interruptions, no saves.

On B's own clock from command reception: entities ready 12.743 ms, preview ready 120.281 ms, commit complete **136.577 ms**, checkpoint complete 136.589 ms, initial world verification complete 194.516 ms. The commit call itself took **0.017 ms** and the completion wait **16.231 ms**. Preview wait was 107.210 ms; verification 57.924 ms, but the two query calls together only 0.090 ms. Waiting for Advances is distinct from synchronous query cost.

A's confirmation took **2,239.214 ms**, longer than 080 despite deferred logging. B's successful command request took 888.295 ms natively, first proof task 821.448 ms, and several later request tasks about one second. Server measured 952.015 ms from A's proof to first delivery, while dispatch handlers stayed below 0.126 ms; that earlier handler measurement excluded accept/startup and writing. No valid inference assigns all remaining time to TCP or Kepler internals. Maximum Advance gaps: A 65.791 ms, B 58.530 ms, none over 100 ms.

## Controlled HTTP comparison: completed 082

See [full phase table, controls and interpretation](../networking/HTTP_LATENCY_080_082.md).

Fourteen phases completed: seven configurations, repeated in reverse order. Each active participant/phase issued ten warmups then 50 measured probes, with a requested 50 ms pause. A/B/P roles generated **480/600/120** probes, **1,200 total** (200 warmups and 1,000 measured). A/B tasks **552/552 and 646/646**; all bodies verified; Python sandbox injection true. Server accepted 1,200 probe connections, no unmatched or still-open probe connections, no bad records. A, B, Python and server all completed. No game mutations or saves.

Earlier aborted attempts are excluded from this completed run. Their final reports were emitted on world deactivation; they did not complete all phases or joins. Restarting only Python against an already advanced server produced Wrong phase order; 082 is not a participant-resume test. Final progress messages mark each phase start/end without per-probe logging.

**Derived:** seconds reside in the native request interval, not worker scheduling, game-query cost, the measured server processing/write sections or task Join. Larger backlog and separate ports do not reliably remove them. A later B-alone repeat remains slow, while the first B-alone phase is fast. Python connects and completes requests quickly inside the Guest sandbox in its sequential comparison phases.

**Hypothesis:** internal request pacing/limiting or another persistent request-path wait could explain the time/order dependence. An approximately 940 ms native call plus a 50 ms pause resembles one request per second, but no fixed limit, limiting scope or Kepler implementation cause is established.

**Unknown:** time before server accept versus delayed completion after response; process-specific filtering, native scheduling and TCP behavior. A Python control is a different process and is not simultaneous with the slow probe on the same socket. No packet capture, native cancellation test, internet route, long polling or persistent-connection test has resolved this yet.

## Next diagnostic direction

Vary request spacing/count and insert a controlled recovery pause, starting each comparison from a defined request history. Include control requests in the accounting. Measure a simultaneously scheduled Python control if needed. If native request delays persist, capture TCP timing to distinguish pre-connect/handshake delay from post-response completion. Keep deduplication, request/task lifetimes and native completion guards; do not bypass an unproven rate limiter or remove commit verification to improve a latency number.

Manual Move/Delete capture, repeated placements, cancellation/Undo ownership, full transforms, arbitrary resources, remote relay security, lobby/menu integration, automatic durable reconciliation and full simulation remain separate milestones.
