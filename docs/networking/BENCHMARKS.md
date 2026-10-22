# Transport and editor measurements

**Updated 2026-10-22.** [Transport](TRANSPORT.md) · [Evidence ledger](../research/DIAGNOSTICS_035_049.md)

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

## Pending: diagnostic 050

Code supplied to the owner, test/result not yet reviewed: warmup 1 plus three rounds of 50/100/250/500 = **2,701 planned rocks and 13 planned commits**, with 26 planned world checks. A target count is not a passed result.

050 compares equal group sizes at increasing population. It also grows undo history and retained diagnostic references; camera/visible content and synchronous world checks can affect frame timing. Three repetitions and one rock resource do not prove general large-park scalability. Preserve approximate trace intervals and Advance counts alongside the game-clock values.
