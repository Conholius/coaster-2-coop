# Own native bridge, local messaging and diagnostics

**Updated 2026-10-24. Runtime evidence: diagnostic 084 through Step 11, plus the separate Lua-logger test. Step 12 is prepared; no reviewed result yet.**

[Current status](../research/STATUS.md) · [083–084 evidence ledger](../research/DIAGNOSTICS_083_084.md) · [Historical Kepler measurements](HTTP_LATENCY_080_082.md) · [Benchmarks](BENCHMARKS.md)

## Current decision

**Derived engineering decision:** stop using the game's Kepler request path as the preferred transport for the next co-op prototype. Its observed history-dependent, approximately one-second request delays are unsatisfactory for responsive editing. Build the game's local communication boundary and diagnostic output in our own DLL, with an external companion responsible for message routing and eventually a separately implemented remote connection.

This is a promising, executed local transport prototype, not finished internet netcode. HTTP itself has not been disproved. A future companion could use HTTP, WebSockets or another appropriate remote protocol without using RunKeplerRequest. No such remote link is verified here.

The earlier HTTP tests remain useful: editor calls, application acknowledgements, command identities, duplicate suppression, object mappings and recovery semantics do not depend on Kepler.

## Why Kepler was set aside: 082 and 083

**Verified in a running game:** completed 082 compared 1,200 probes and localized long waits inside the native request interval. Server handling and a separate Python control stayed fast. It did not establish the internal cause.

083 ran four independent request-spacing experiments: 50, 250, 500 and 1,000 ms. Each completed 160 pre-pause probes, a 120-second pause with no diagnostic HTTP calls, and 60 post-pause probes. All 220 bodies per run were correct, all 224 tasks per run joined, and every run completed.

| Requested spacing | Pre-pause requests above 900 ms | Last 20 pre-pause native-call median | Post-pause requests above 900 ms |
| --- | ---: | ---: | ---: |
| 50 ms | 54 / 160 | 1,002.679 ms | 0 / 60 |
| 250 ms | 26 / 160 | 1,000.346 ms | 0 / 60 |
| 500 ms | 0 / 160 | 13.192 ms | 0 / 60 |
| 1,000 ms | 0 / 160 | 18.303 ms | 0 / 60 |

Source: interval-labelled private 083 logs, GROUP / MEASURE records for PRE and POST, RESULT and completion markers. Requested spacing is not identical to the actual start gap when a request itself takes longer.

**Hypothesis:** internal history-dependent pacing or throttling is consistent with these results and recovery after a pause. Exact quota, refill policy, scope and implementation remain unknown. The finite 500/1,000-ms runs do not establish indefinitely sustainable request rates. Do not describe this as a proven Frontier server policy, a general HTTP limitation or a mechanism to bypass.

## Executed local architecture

~~~mermaid
flowchart LR
    A["PC2 Lua A"] <--> DA["Own DLL A"]
    DA <--> PA["Named Pipe A"]
    PA <--> P["Python outside sandboxes"]
    P <--> PB["Named Pipe B"]
    PB <--> DB["Own DLL B"]
    DB <--> B["PC2 Lua B"]
~~~

**Verified in a running game:** Step 9 used a game client and Python in one sandbox. Step 10 crossed from the sandboxed Host to external Python. Step 11 used two simultaneous game clients, in PC2CoopHost and PC2CoopGuest, and external Python.

The native worker transfers copied data only. Lua and game APIs stay on the game's calling thread. Lua enqueues a bounded message and retrieves completed incoming messages in later Advances; the worker handles Windows pipe I/O. The tested transport uses length-prefixed frames, a 4,096-byte frame cap, fresh sessions, roles and sequence checks, bounded queues, cancellation events and finite operation timeouts.

Each client has its own endpoint. The Python relay handles the two endpoints independently and coordinates its shared state. In Step 11 the artificial delay withheld one forwarded message; it did not sleep inside either client's relay handler.

Named Pipes are a same-machine boundary. They do not themselves connect two computers. The external companion's future internet transport, authentication and deployment remain separate work.

## Lua interface and compatibility boundary

Steps 1–5 progressively established loading, own-file logging, repeated callbacks, echoing and explicit native buffer copies. Early load/ABI experiments included crashes; the successful progression does not make arbitrary exported functions or standard Lua binaries interchangeable.

Steps 6/7 passed direct input-string reads and direct Lua-string returns, including empty text, embedded null bytes, UTF-8 and rejected invalid inputs. The return path uses the game's own profiled string routine with executable/code checks, stack-bound checks, result validation and native-source-buffer clearing. package.loadlib availability alone is not a compatibility guarantee.

The tested native profile is tied to the owner's Windows x64 PC2 executable. Binary hashes and raw implementation research stay in private evidence. Unsupported builds must fail the profile checks rather than reuse guessed layouts or offsets. No general compatibility across game updates is claimed.

### Step 8: endurance and garbage collection

Private Native084-Step8Endurance.log and matching game-log summaries report:

- 4,100 verified direct string round trips, 4,107 returned native calls.
- Five phases: 100 warmup, then 1,000 each baseline, allocation pressure, forced GC and recovery.
- 13 full collections, 13 weak-reference control objects collected, 200 allocation bursts.
- 6,662 retained-answer checks without changed strings; final retained audit 0.4097 ms.
- NativeCompleted=true and LuaCompleted=true.
- Full GC median 5.7515 ms, maximum 8.5825 ms.
- Lua-call median 0.0287–0.0333 ms across phases, maximum 0.1643 ms.
- No Advance gap above 33 ms in the forced-GC phase; aggregate maximum 61.3321 ms occurred elsewhere.

**Derived:** direct returned strings survived the tested allocation/GC pressure. This is not proof against all memory bugs, all GC states or longer production workloads.

## Timing boundaries and measurements

All measurements are owner-run local diagnostics, not a benchmark of internet play.

| Test | DLL-worker exchange median / P95 | Lua-observed round trip median / P95 |
| --- | --- | --- |
| Step 9, one sandbox | 0.1144 / 0.1745 ms | 12.2530 / 13.7061 ms |
| Step 10, Python outside sandbox | 0.1030 / 0.1602 ms | 11.5627 / 12.2336 ms |
| Step 11, A | 0.1184 / 0.3824 ms | 14.9883 / 31.2429 ms |
| Step 11, B | 0.1211 / 0.4592 ms | 21.6822 / 24.5609 ms |

Steps 9/10 measure Python echo completion. Step 11's Lua round trip includes delivery to the other game, content verification there and a returned application-level acknowledgement. Its DLL-worker metric includes individual relay requests/polls; it is not the full A-to-B-to-A interval.

Advance scheduling, the relay's poll interval and the target client's processing contribute to Lua latency. Step 11 measured normal relay queue residence at median 5.3057 ms / P95 9.7200 ms. Normal full confirmation maxima were 168.9374 ms on A and 39.7946 ms on B; summaries do not identify a unique cause for every outlier.

Comparing a 0.1-ms pipe-worker exchange against an older end-to-end Kepler measurement would be unfair. The evidence supports a responsive local path in these tests, not a universal speedup over every fast Kepler request.

### Artificial two-second delay

| Test | Advances while waiting | Maximum monitored Advance gap while waiting |
| --- | --- | --- |
| Step 9 | 158 | 17.4790 ms |
| Step 10 | 174 | 13.1461 ms |
| Step 11 A | 146 | 22.6718 ms |
| Step 11 B | 175 | 13.3105 ms |

In Step 11 Python held A message 101 for 2,001.2016 ms. During that interval **112 reverse-direction B-to-A messages were verified and acknowledged**. A's delayed full confirmation took 2,018.0314 ms. Both directions otherwise completed 1,000 verified messages each.

Step 9/10 aggregate Advance maxima of 196.1418 / 191.0968 ms occurred outside the artificial delay windows. Step 11 aggregate maxima were 61.0344 ms on A and 39.8484 ms on B. Advance gaps are callback observations, not presented-frame measurements; these records do not justify a blanket no-stutter claim.

## Own logger: independent of ACSEDebug

The separate logger test passed with ACSEDebug present and, per the owner's test setup, removed. Each run wrote exactly 1,000 ordered test records, plus progress and summary records (1,014 queued records total). File-content review found no missing, duplicate or altered test messages. UTF-8, escaped null bytes, empty payloads and long text matched the fixtures. Both logs end with LuaCompleted=true and NativeCompleted=true; workerError=0 and queue-busy count=0.

Lua enqueue median was 0.0166 ms in each run. The DLL copies messages into a bounded queue; regular file writes happen on a worker. Profile checking and opening the file still occur during initialization. Flush and close results remain part of finalization.

The Lua-completion footer records the successful Lua finalization request; it is not a second independent observation that Lua received the later file-close acknowledgement. The latter is a separate return to Lua. Mode labels alone do not detect whether ACSEDebug is installed.

**Verified scope:** our own Lua diagnostic output works without using ACSEDebug's logger, and Step 11's two-client run uses the own logger. ACSE remains part of the mod-loading setup. This is a diagnostic logger, not a complete interactive debugger, a capture of all internal engine messages or a native crash-dump replacement. A failure before the DLL loads still needs a fallback/bootstrap diagnostic path.

## Step 11 completion evidence

Sources are the two role-labelled Native084-Step11.log files and Python084-Step11.log, with matching private session identifiers.

| Counter | A | B |
| --- | ---: | ---: |
| Unique data sent / peer verified acknowledgements | 1,000 / 1,000 | 1,000 / 1,000 |
| Received and checked peer data | 1,000 | 1,000 |
| Lua command queue accepted | 2,001 | 2,001 |
| Frames delivered from DLL to Lua | 2,006 | 2,006 |
| Wire requests and replies each | 6,679 | 6,668 |
| Own logger accepted / written | 23 / 23 | 22 / 22 |
| Worker error | 0 | 0 |

Both Lua/native completion markers and PythonCompleted are true. Python reports sandboxieInjected=false and both clients closed. The wire count includes transport polling/control frames; it is not the number of unique game messages. Python verifies payload and sequence; the receiving game verifies the message before acknowledging it. No park edits or saves occurred in Step 11.

## Step 12: prepared, not yet verified

Next: capture one manually placed AQ_Rock_04 on A, leave its distinct next preview active, transfer resource plus position/orientation through the DLL/companion path, create and commit once on B, verify mappings/world state over later Advances, then redeliver identical command bytes and require no additional effect.

The supplied test limits scale to one, uses logical command/object IDs separate from local Part/Entity IDs and explicit applied/duplicate proofs. Its code and instructions have been provided, but no Step-12 result was reviewed at this documentation cutoff. The older 079–081 manual-Create success used HTTP; it must not be relabelled as native-pipe Create success.

## Remaining engineering risks and work

- Port command/status/revision, origin filtering and recovery guards to this transport; earlier HTTP recovery tests do not automatically certify pipe recovery.
- Test pipe break/reconnect, external companion restart, slow/unavailable peer, bounded-queue overflow/backpressure and world deactivation during outstanding I/O.
- Investigate the owner's recurring game-exit crash reporter. It predates this DLL work and persists in the reported setup without ACSEDebug; cause is unknown and a controlled no-mod / ACSE-only / own-DLL comparison has not passed. In-test success does not prove a crash-free process exit.
- Test the next manual Create, then Move/Delete and guest-originating edits, conflicts and Undo ownership.
- Implement and test an authenticated remote companion-to-companion or relay connection. Define baseline/park transfer, lobby and saving independently.
- Verify durable status/checkpoint integration, ambiguous crash outcomes, mismatched saves, late join and repair.
- Expand resources, rotation/scale/property coverage, long soaks and actual presented-frame analysis.

No shipped netcode, arbitrary-crash exactly-once, internet co-op or full-park simulation synchronization is claimed.
