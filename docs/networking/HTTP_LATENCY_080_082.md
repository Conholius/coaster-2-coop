# HTTP latency: diagnostics 080–082

**Reviewed 2026-10-23.** [Status](../research/STATUS.md) · [Diagnostic ledger](../research/DIAGNOSTICS_069_082.md) · [Transport contract](TRANSPORT.md) · [Benchmarks](BENCHMARKS.md)

## What is established

**Verified in a running game:** two local PC2 clients can replicate one player-confirmed AQ_Rock_04 placement while the next host preview stays active. The guest applies once and suppresses an identical delivery. This is not yet a shipped co-op mod, a remote latency budget or a full-park synchronization test.

**Verified measurements:** 080's host-observed completion was 591.284 ms; 081's was 2,239.214 ms. Neither is the native guest commit duration. In 081, command receipt to completed guest commit was 136.577 ms; the commit invocation and completion wait together about 16.25 ms. Different runs/loads must not be treated as an isolated logging A/B experiment.

**Current finding:** completed 082 localizes the large waits to the RunKeplerRequest interval. Larger listener backlog and separate ports do not reliably remove them. A Python control inside the Guest sandbox has no seconds-long delays. This narrows the suspect path; it does not prove where inside Kepler/TCP the wait occurs or a fixed request-rate limit.

## 080 and 081: separate action from proof

080 did the following serially: observe original host commit, wait for the next preview twice, verify the committed object across two Advances, publish, submit an additional host proof, let B poll/deliver/apply/verify, then let A poll the application proof. The relay withheld delivery until A's initial proof arrived.

| 081 local clock / operation | Measured ms |
| --- | ---: |
| A commit return to component observing captured data | 64.921 |
| A additional next-preview wait | 113.893 |
| A initial world verification including two Advance waits | 37.756 |
| A publication task | 20.637 |
| A commit return to guest application confirmation | 2239.214 |
| B entity wait | 12.536 |
| B prepared transform and validation | 0.221 |
| B preview completion wait | 107.210 |
| B CommitPreview invocation | 0.017 |
| B commit completion wait | 16.231 |
| B checkpoint invocation / idle wait | 0.006 / 0.001 |
| B two-Advance verification | 57.924 |
| B two native world-query bundles combined | 0.090 |
| B command receipt to completed commit | 136.577 |
| B command receipt to completed initial verification | 194.516 |

The two clients' performance clocks are independent. Durations above are meaningful locally; do not subtract their absolute timestamps. The first successful world query is not proof of the earliest safe query time. Native completion waits include engine work and time until the component observes completion; they are not pure CPU timings.

Derived optimization candidates: publish a safely observed committed object before ancillary proof gates, run additional audits outside the delivery-critical path, replace short polling with a validated bounded long poll, and investigate connection reuse. None has been performance-validated here. Do not skip native completion, revision/deduplication or unresolved-outcome handling.

## 082 method

Original diagnostic files: server082.py (TimedServer, Handler.setup/do_GET, probe/control routes and final summaries), control082.py (raw socket probe), and pc2coopdiagnosticcomponent.lua (request worker, phase coordinator, probe validation, Advance and final report).

- Three loopback probe listeners: queue size 5; queue size 128; a separate queue-128 listener. A fourth listener handles phase barriers/control.
- Fourteen phases: seven configurations then the reverse order, with all three participants acknowledging every phase.
- Per active participant/phase: ten warmups, 50 measured requests and a requested 50 ms pause between requests. Exactly one native request in flight per game client.
- Native control/coordination requests are additional traffic, though on a separate listener. An inactive game participant can have a pending control request; "A only" means only A issues probe requests, not that the other game process is absent.
- Plain HTTP/1.0 with Connection: close, fresh requests, fixed 256-character payload plus identity and timing fields; no compression or connection reuse.
- Every reply validates session, phase, role, sequence, request ID and full payload. Probe failures are counted, not automatically retried.
- No editor calls, database queries, previews, commits or saves. Timings are retained in memory and printed after completion; phase progress only is printed during the run.
- Python control runs inside the configured Guest sandbox; SbieDll injection is checked. That check alone does not independently identify the exact box name or certify complete isolation.

The completed run produced A 480, B 600 and Python 120 probes: 200 warmups plus **1,000 measured probes**, **1,200 total**. A joined 552/552 tasks; B 646/646. Server: 1,200 valid successful probe connections, no bad, unmatched or open probe connections. Every participant and server reported Completed=true. Earlier incomplete runs are excluded.

## Completed 082 phase results

Medians include native request creation, worker dispatch, RunKeplerRequest and task joining. The slow columns count measured requests above 900 ms out of 50, excluding warmups.

| Phase | Probe configuration | A median ms | B median ms | Python median ms | Above 900 ms A / B / P |
| --- | --- | ---: | ---: | ---: | --- |
| 1 | Forward A only, queue 5 | 15.855 | — | — | 0 / — / — |
| 2 | Forward B only, queue 5 | — | 23.810 | — | — / 0 / — |
| 3 | Forward A+B shared, queue 5 | 20.341 | 24.203 | — | 15 / 15 / — |
| 4 | Forward B only, queue 128 | — | 947.201 | — | — / 50 / — |
| 5 | Forward A+B shared, queue 128 | 15.762 | 946.445 | — | 0 / 50 / — |
| 6 | Forward A+B separate, queue 128 | 14.535 | 944.928 | — | 1 / 50 / — |
| 7 | Forward Python Guest only, queue 128 | — | — | 0.989 | — / — / 0 |
| 8 | Reverse Python Guest only, queue 128 | — | — | 0.936 | — / — / 0 |
| 9 | Reverse A+B separate, queue 128 | 20.477 | 945.581 | — | 5 / 50 / — |
| 10 | Reverse A+B shared, queue 128 | 25.338 | 944.326 | — | 8 / 50 / — |
| 11 | Reverse B only, queue 128 | — | 946.516 | — | — / 50 / — |
| 12 | Reverse A+B shared, queue 5 | 13.142 | 944.800 | — | 0 / 50 / — |
| 13 | Reverse B only, queue 5 | — | 946.212 | — | — / 50 / — |
| 14 | Reverse A only, queue 5 | 14.401 | — | — | 0 / — / — |

Phase 3 is bimodal: a small median conceals a slow tail. A/B p95 totals there are 959.362 / 950.166 ms. In Phase 4 B's native run mean is 941.488 ms and total mean 946.332 ms. A is not immune: some shared/separate phases contain long requests, although later A-only and some shared phases are fast.

## Server and Python measurements

For game probes, mean accepted-connection-to-reply-ready time per group was about 0.4–1.2 ms, with maximum 10.8213 ms. Response headers/body write and flush maxima stayed below **0.42 ms**. These are local server durations, not client-receipt timestamps and not time spent before accept. A successful flush does not prove the remote application has consumed the response.

| Python Guest control | Forward | Reverse |
| --- | ---: | ---: |
| TCP connect median ms | 0.22105 | 0.18860 |
| TCP connect p95 ms | 19.4532 | 16.9406 |
| TCP connect maximum ms | 23.6344 | 23.5889 |
| Send to first response byte median ms | 0.51905 | 0.50175 |
| First byte to full body median ms | 0.02130 | 0.02270 |
| Total median ms | 0.98940 | 0.93645 |
| Total p95 ms | 19.8376 | 17.3128 |
| Total maximum ms | 24.0285 | 24.0407 |

Python's total includes creating the socket and validating receipt timing boundaries. It is a different process, uses direct sockets without proxy discovery, and runs in sequential dedicated phases, not concurrently with every slow Kepler probe. Therefore it makes a universal sandbox/TCP/server fault less likely but cannot exclude process-specific network filtering, native proxy behavior, or a history-dependent condition.

Native worker queueing is about 0.17–0.20 ms, request preparation generally microseconds, and native return-to-Join about 5 ms. The approximately 940 ms waits are inside RunKeplerRequest. Server accept/thread/header/handler/write times do not account for them, but time before accept and response consumption after flush remain unmeasured by the server.

Advance gaps: A max **23.9535 ms**, none above 33 ms; B max **56.5462 ms**, three above 50 ms, none above 100 ms. Seconds-long HTTP calls therefore do not correspond to seconds-long component Advance gaps in this run. Advance gaps are not presented-frame timings or a guarantee of visual smoothness.

## Interpretation and next experiments

**Derived**
- The small queue alone is insufficient: B remains slow alone and with queue 128, separate ports, and on the reverse queue-5 test.
- There is a request-history/time dependence: initial single-client probes are fast; delay starts during Phase 3 and persists on B.
- Synchronous Lua preparation, world verification, server dispatch/write and post-return Join cannot explain a roughly one-second request interval.
- HTTP and Kepler must not be treated as synonymous. These tests have not shown that HTTP itself is a dead end.

**Hypotheses, not findings**
- Native request pacing, rate limiting, a shared process/account service queue, or another internal wait could fit the pattern.
- Roughly 940 ms inside the native call plus the 50 ms requested pause resembles approximately one request/second. This is not a measured fixed quota, a statement about Frontier's server policy, or permission to evade a limit.
- Process-specific sandbox/network behavior or delayed native completion remains possible.

**Unknown**
- Whether the dominant wait precedes TCP handshake/HTTP send or follows the network response.
- TCP retransmissions, exact native throttling scope/window, proxy discovery, connection pooling and process/account coupling.
- How results change over a remote HTTPS route or without two local game processes.

**Next test proposal, not executed:** use a controlled request history, vary request spacing (for example 50/250/500/1000 ms), include a no-request recovery interval, and count coordination/background request exposure where observable. Add a contemporaneous Python control and, if necessary, a restricted TCP trace. Distinguish first-byte/full-body arrival from native completion before naming a cause.

A sidecar, long polling, persistent connections, adaptive polling and bounded batching remain alternatives to test, not validated fixes. Reducing the number of proofs on the delivery-critical path can reduce application latency but must not conceal the native-request delay or weaken replay safety.

## Sources and reproducibility

Primary evidence: owner-retained matched 080/081 A/B logs; completed 082 A/B RESULT, GROUP, MEASURE, SLOW_OR_FAILED and ADVANCE_GAPS records; server FINAL_META/SERVER_GROUP/SERVER_SAMPLES/CONNECTIONS; sandboxed Python RESULT/GROUP. These are summarized rather than published raw. The completed 082 server reports Python 3.11.0; no public executable/package hash is claimed.

Protocol and method references: [Python 3.11 socketserver request_queue_size](https://docs.python.org/3.11/library/socketserver.html#socketserver.BaseServer.request_queue_size), [ThreadingHTTPServer](https://docs.python.org/3.11/library/http.server.html#http.server.ThreadingHTTPServer), [HTTP long polling](https://www.rfc-editor.org/rfc/rfc6202.html), [HTTP/1.1 persistent connections](https://www.rfc-editor.org/rfc/rfc9112.html#section-9.3). These explain mechanisms, not the cause of the observed PC2 delay.
