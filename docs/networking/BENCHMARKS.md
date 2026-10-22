# Transport and editor measurements

**Updated 2026-10-22.** [Transport](TRANSPORT.md) · [Earlier ledger](../research/DIAGNOSTICS_035_049.md) · [050–068 ledger](../research/DIAGNOSTICS_050_068.md)

All numbers below are owner-run local tests on the reported PC2 setup, not public automated two-client QA. Read timing boundaries before comparing columns. Raw logs and personal machine details stay private.

## HTTP downloads: diagnostics 046 and 047

046: two warmups, then five samples for each ascending size (27 requests). The corrected offline log is the timestamped offline repeat; an earlier file labelled NoInternet actually contained diagnostic 043 and was excluded. Both valid 046 runs passed full body checks and task joins. Even the run called "with internet" performed its 27 requests while Frontier was disconnected; internet availability and Frontier connection are different observations.

047: after readiness, 30-second settling period, ten mixed-size warmups then 20 balanced mixed rounds (100 measured requests, 20 per size). Local seed 47047 / plan signature 1049800266; one in-flight request, 0.1-second pause after join. The reviewed run had Frontier connected before the measured requests. All **110/110** requests, bodies, sequences, task joins and reference releases passed; server/client lengths and sizes matched.

Mean **task spawn-to-join** time in ms (includes native request, result verification, scheduling and logging; excludes deliberate post-join pause):

| Payload | 046 internet available | 046 corrected offline | 047 settled / mixed |
| --- | ---: | ---: | ---: |
| 1 KiB | 81.0 | 21.9 | 19.4 |
| 8 KiB | 90.3 | 18.0 | 17.9 |
| 64 KiB | 54.2 | 17.8 | 18.4 |
| 256 KiB | 19.4 | 24.7 | 20.0 |
| 1 MiB | 22.2 | 24.4 | 20.5 |

047 game-clock request means: 8.323 / 6.716 / 7.212 / 8.892 / 9.404 ms in size order. **28/100 measured requests reported zero**; all verification phases reported zero. The game clock is too coarse/frame-dependent to treat these as pure CPU/network timing. Approximate ACSEDebug HTTP_START-to-HTTP_NATIVE_RETURN means: 15.45 / 13.95 / 14.35 / 16.55 / 17.60 ms, including trace overhead. Task P95s were about 22.4–22.8 ms; the largest task outlier was 32.9 ms.

The 047 server's 1 MiB measured mean construction/write calls took about 0.827/0.174 ms. A socket write returning is not proof of remote delivery. No uniform first-half-to-second-half slowdown was observed in this short run.

**Derived:** no strong payload-size bottleneck up to 1 MiB in this local sequential download test. The earlier high small-packet times did not recur after settling/mixed order; startup influence is plausible, not a uniquely isolated cause. No maximum bandwidth, internet latency, concurrent clients, upload performance, long-duration stability or engine edit capacity follows from this test.

## Grouped editor operations: diagnostics 048 and 049

Both used a warmup rock and three mixed rounds of 1/5/10/25/50 rocks: **274 unique parts, 16 previews, 16 commits, 16 checkpoints and 32 successful world checks**. Every group verified all previously created rocks over two Advances, including resource, position and stable primary mapping. No diagnostic save call. These tests did NOT transport the groups via HTTP.

048 waits for each individual CreateNewPart before issuing the next. 049 issues all creates with one shared completion token, waits once and then opens/positions each edit object. Both use one hierarchy and one preview/commit per group.

Mean game-clock times in ms, excluding warmup:

| Rocks | 048 prepare | 049 prepare | Prepare Advances 048 -> 049 | 048 pipeline | 049 pipeline |
| --- | ---: | ---: | --- | ---: | ---: |
| 1 | 11.599 | 11.698 | 1 -> 1 | 46.866 | 47.165 |
| 5 | 58.440 | 11.562 | 5 -> 1 | 93.803 | 46.608 |
| 10 | 116.613 | 11.565 | 10 -> 1 | 152.129 | 46.684 |
| 25 | 291.224 | 11.505 | 25 -> 1 | 326.238 | 46.567 |
| 50 | 581.525 | 11.452 | 50 -> 1 | 616.408 | 46.371 |

Prepare includes hierarchy, creation requests/entity waits, StartEditingPart_Scenery, positioning/readback and change-list construction. Pipeline adds completed preview, committability/ID checks, completed commit/undo idle and Checkpoint. Full world checks and the one-second inter-group pause are outside it.

049's measured groups all needed four pipeline Advances: one preparation, two preview, one commit. Preview remained about 23 ms and commit about 12 ms. For 50 rocks the game-clock pipeline was about 13.3 times shorter than 048. This is a reduction in elapsed waits, not proof of 13.3x less CPU work.

049 reports zero game-clock time for ISSUE and EDIT, illustrating under-resolution of same-frame synchronous work. Approximate trace intervals show 21.7–22.3 ms preparation and 64–67 ms pipeline, including additional trace messages. The large Advance reduction and trace timings corroborate the improvement; exact speed ratios should not be overinterpreted.

**Verified:** shared completion tokens and a grouped commit worked for the tested rock up to 50 per group. **Derived:** 048's linear time was strongly influenced by our per-object waiting strategy; it is not a measured hard engine limit. No growing commit cost was detected in this range. Concurrent CPU execution of the creation requests has not been demonstrated by this test.

## Growing park, timer and verification: 050–054

All successful runs created 2,701 AQ_Rock_04 parts with 13 commits/checkpoints. Warmup 1; measured sizes 50/100/250/500 repeated three times. Undo history and retained test references grew together with park population; the experiment does not isolate object count alone.

050/051's frame-dependent game clock gave roughly 47 ms pipelines even for larger groups. Minimal own logging in 051 did not produce a strong change on that clock; this does not isolate logging overhead or prove synchronous work is free. 052 added `GetPerformanceTimer`, `DiffPerformanceTimers(later, earlier)` and `PerformanceTimeToMilliseconds` plus Advance-entry monitoring. These measure elapsed time, not thread CPU time or presented frame times.

| Group size | 052 pipeline mean ms | 053 new-group check elapsed mean ms | 054 pipeline mean ms |
| --- | ---: | ---: | ---: |
| 50 | 48.0373 | 23.7407 | 48.2038 |
| 100 | 50.1526 | 24.1508 | 50.1057 |
| 250 | 57.9007 | 24.6733 | 56.0074 |
| 500 | 67.1757 | 27.0821 | 67.3099 |

052 checked all accumulated parts: 33,726 individual checks. 053 checked each new part twice: 5,402, reducing mean own VERIFY callback work from about 3.255 to 0.572 ms and maximum from 7.254 to 1.885 ms. Pipeline time itself did not show a comparable improvement; these are different workload components.

054 added a final complete inventory to new-group checks: 8,103 total checks. Final 2,701-object audit passed resource, world position, mapping and unique-primary checks: **8.7688 ms work**, **19.7844 ms elapsed**, one Advance. Its following gap was 20.3409 ms. Maximum recorded gap overall was 26.8199 ms, none above 33 ms. This is one-pass final auditing, not the two-pass distributed audit in 067.

## Synthetic registry/history comparison: 066

Objects: 1,000 / 10,000 / 50,000 / 100,000. Histories independently: 1,000 / 10,000 / 100,000. All **12 combinations / 24 layouts** passed. Both variants keep canonical command content and DONE status; minimal objects keep part ID/revision/deletion state. FULL adds resource/transform/entity snapshots and detailed action data. Extra update/verification work is therefore not a pure table-layout comparison.

Largest case: 100,000 synthetic objects and 100,000 history entries. **Summed measured work**, excluding deliberate inter-chunk Advance waits:

| Stage | MINIMAL ms | FULL ms |
| --- | ---: | ---: |
| Build objects | 285.819 | 400.304 |
| Build history | 348.544 | 512.006 |
| 6,144 lookups | 4.421 | 5.009 |
| 6,144 updates | 4.147 | 8.702 |
| 4,608 classifications | 2.579 | 2.853 |
| Remove 25,000 old entries | 19.218 | 18.053 |
| Full final verification | 347.043 | 401.534 |

Each variant verified 1,536 duplicates, 1,536 ID conflicts and 1,536 unknown commands. Pruned old sequences are rejected below a modeled checkpoint boundary; exact old-content comparison is no longer claimed. This is not durable journal compaction.

There was one monitored gap above 50 ms (51.529 ms during FULL object building) and maximum own callback 20.316 ms during MINIMAL building. Lua heap readings cover the whole VM, automatic GC and temporary data; they are not isolated retained size or process RAM. No forced collection. Setup/transition monitoring was less complete than 067/068, so do not claim absence of all unmonitored stalls.

## Real-object registry, native queries and Moves: 067

Populations: 100 / 500 / 1,000 / 2,701. All 2,701 rocks registered; 72 Move groups, 1,464 individual edits, 84 commits/checkpoints, 108 probe samples and 180,370 world checks passed. Final positions/revisions matched and no mapping changes occurred. All 84 entity groups / 168 native operations completed; no save.

Average microseconds per probe item (1,536 items per name/population), including Lua loops, binding and checks, not isolated native function CPU:

| Probe | 100 objects | 500 | 1,000 | 2,701 |
| --- | ---: | ---: | ---: | ---: |
| Control fixture access | 0.2169 | 0.2353 | 0.2259 | 0.2289 |
| Minimal registry | 0.4807 | 0.5885 | 0.6113 | 0.6383 |
| Part exists | 0.2947 | 0.3271 | 0.3040 | 0.3393 |
| Primary entity | 0.3286 | 0.4093 | 0.4201 | 0.4560 |
| All entities + compare | 1.8169 | 1.9753 | 1.9952 | 2.0795 |
| Resource + compare | 0.5251 | 0.5727 | 0.5719 | 0.6283 |
| World transform + compare | 1.7832 | 2.2299 | 1.9351 | 2.0840 |
| Exists → fresh primary → position | 1.9533 | 1.9429 | 2.4586 | 2.6711 |
| Full object/mapping check | 5.4569 | 6.7883 | 6.0564 | 5.6401 |

Repeated fixed probes emphasize warm access, not every real-world cache pattern. Simple-rock results do not establish complex-object costs. A control is a reference; subtracting it does not magically isolate the exact native call cost.

For 50-object moves (six samples/population), pipeline elapsed means stayed 46.7252 → 46.8460 ms from smallest to largest park. At the largest park register updates averaged **0.0219 ms per group**.

Two-pass verification work after 50-object moves:

| Population | Changed 50 work ms | All-object work ms | All-object elapsed ms |
| --- | ---: | ---: | ---: |
| 100 | 0.7880 | 1.1367 | 69.2223 |
| 500 | 0.7526 | 6.0554 | 211.7438 |
| 1,000 | 0.5512 | 12.9144 | 401.1679 |
| 2,701 | 0.7445 | 36.5506 | 1,047.5181 |

The one-second audit is spread over chunks/Advances, not one frozen frame. 067 recorded a 36.005 ms own callback / 61.1868 ms maximum gap in aggregated MIXED_TRANSITION, without pairing those maxima to the same event. Startup gap reached 242.2739 ms; source attribution was incomplete. Read 068 before drawing a cause.

## Paired section profiling: corrected 068

Same game workload/counts as 067, no added yield points. A first name-shadowing error stopped before any objects/commits; excluded. The corrected run completed and paired **6,867 callbacks** with following gaps; section/gap accounting passed, no section overflow. Twenty longest callbacks and twenty largest gaps were retained separately for STARTUP and WORK. Chronological traces keep their first 32 spans; omitted trace spans are still included in section totals.

| Paired event | Own callback ms | Following entry gap ms | Outside measured callback ms |
| --- | ---: | ---: | ---: |
| Startup catalogue result handling/search | 37.2776 | 49.1728 | 11.8952 |
| Slowest WORK callback, registry + next Create group | 3.4665 | 16.0869 | 12.6204 |
| Largest WORK gap, 250-part edit/preview preparation | 2.9393 | 21.6808 | 18.7415 |

The startup catalogue section accounts for **37.2313 ms**. Its completion-status poll accounts for only 0.0032 ms in that callback. The broader label still includes `GetAllResults` and the Lua resource search, so their individual contributions remain unknown. This is a concrete one-time startup candidate, not proof of an expensive steady-state placement query or retrospective proof of 067's spike cause.

The slowest WORK callback combined prior-group register creation (1.3812 ms), 250 new native create requests (1.2063 ms total), plan construction (0.5566 ms) and other work. Final register checking occupied 2.8852 ms in another callback. Native immediate `CommitPreview` / `Checkpoint` traced-call maxima were 0.0881 / 0.0139 ms, excluding later asynchronous engine work.

Across WORK: own maximum **3.4665 ms**, gap maximum **21.6808 ms**, none over 33 ms. Startup gap maximum **237.2709 ms**, almost entirely outside the measured callback. No attribution of that outside time to a specific engine subsystem, service, scheduler or GC is established.

Profiling has overhead: 256 direct/traced no-op calls measured 0.0048 / 0.2742 ms; nothing was subtracted. The instrumented run cannot prove the earlier 067 outliers are fixed or will never recur. Timed sections are exclusive wall-time inside the measured callback; they include instrumentation/preemption and are not pure native CPU time. Callback-return tail bookkeeping may be included in the reported outside interval. Presented-frame capture remains open.

## Derived direction, not a shipped performance guarantee

Prefer a minimal registry, native lookups for changed objects, bounded mutation groups and occasional distributed full audits. Initial builds, serialization, allocation and catalogue materialization/search deserve separate budgets. Do not continuously scan/retransmit the park, extrapolate synthetic capacity to real objects, remove completion/verification waits, or merge dependent transactions merely to lower Advance counts. None of these local measurements establishes internet throughput, full simulation synchronization or a universal frame-time budget.
