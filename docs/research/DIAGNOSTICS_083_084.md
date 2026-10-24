# Diagnostics 083–084: transport change and own diagnostics

**Reviewed 2026-10-24.** [Current status](STATUS.md) · [Detailed architecture, timings and limits](../networking/NATIVE_BRIDGE.md)

Evidence below comes from owner-run game tests and reviewed private logs. No game test was executed for this documentation update. Raw logs, local user paths, binary hashes, DLL/OVL packages and extracted game code remain private.

| Diagnostic / private source anchor | Verified finding | Boundary |
| --- | --- | --- |
| 083 interval-labelled game logs, PRE/POST GROUP and MEASURE, RESULT/completion | Four spacings, 220 correct probes and 224 joined tasks per run; 50/250-ms runs develop one-second native tails, 120-second quiet period restores fast post samples | Exact limiter, quota and sustainable rate unknown; HTTP protocol not disproved |
| 084 Steps 1–5 own native logs and matching Lua logs, previously reviewed | Loading/callbacks, own-file path/logging and explicit native copies established incrementally after failed early attempts | Presence of loadlib alone is insufficient ABI proof |
| Step 6 Direct and Step 7 Return paired logs | Direct strings in/out, invalid-input rejection, profile/stack checks, source-buffer clearing and result checks pass | One profiled x64 PC2 build; not arbitrary Lua ABI or future version support |
| Step 8 Endurance native and game summaries | 4,100 round trips, 13 confirmed full GCs, 6,662 retained checks, both completion markers true | Finite soak; Advance observations are not render-frame telemetry |
| Step 9 Pipe paired game/native and Python output | 1,010 echoed data messages, three Python notices acknowledged, same-sandbox pipe, 158 Advances during deliberate wait | One game client; not sandbox-boundary or two-game proof |
| Step 10 Pipe native/game/Python logs | External Python reports no SbieDll injection; same counts pass; 174 Advances during wait; all completions true | Same Windows machine, one game client |
| Separate LuaLog With/WithoutACSEDebug files | File review confirms 1,000 correct test records / 1,014 queued records each; workerError=0, both completion markers true | Owner disabled ACSEDebug; label alone is not detection; own logger is not a full debugger |
| Step 11 two Native logs plus Python summary | 1,000 messages per direction verified and acknowledged, own logger, matching sessions/counters, all completions true; 112 reverse ACKs during A-to-B delay | Two local sandboxed clients and external local companion; no game mutation, WAN or pipe-recovery proof |
| Step 12 code/instructions | Manual host Create over native bridge, guest one-time application and duplicate proof prepared | **No reviewed runtime result at this cutoff** |
| Owner report after successful Step 11 | Crash reporter on game exit, reportedly predating native work and persisting without ACSEDebug | No isolated root cause or controlled shutdown fix |

## Reproducibility summary

Use copied/disposable test installations and tropical test parks. Steps 10/11 run the companion outside Sandboxie under the same Windows user; each game has its own permitted pipe endpoint. Step 11 waits for both roles and a fresh session, tests exact payloads/sequences, synchronizes after 100 acknowledged messages each, holds only A message 101, and requires final counts and orderly channel closure.

Read the [native bridge page](../networking/NATIVE_BRIDGE.md) for measurement boundaries. The proposed product can use a host/join workflow; independently loaded local test parks are diagnostic scaffolding, not a final UX requirement. Source-level/profile observations are not universal build compatibility.

## Interpretation

**Derived:** the own local DLL/pipe path is promising and now preferred over Kepler for continued experiments. **Hypothesis:** Kepler history-dependent throttling/pacing. **Unknown:** production remote networking, arbitrary crash recovery and exit-crash root cause. Earlier HTTP scenery and recovery findings remain evidence for their original tested transport/lifetime only.
