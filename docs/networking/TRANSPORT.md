# Transport: own native bridge and historical Kepler HTTP

**Updated 2026-10-24; native bridge through 084 Step 11, Step 12 pending.** [Two-client ledger](../research/DIAGNOSTICS_069_082.md) · [Native/Python latency comparison](HTTP_LATENCY_080_082.md)

[Current status](../research/STATUS.md) · [035–049 evidence ledger](../research/DIAGNOSTICS_035_049.md) · [Proposed synchronization](HTTP_SYNC_DESIGN.md) · [Measurements](BENCHMARKS.md)

## Current transport choice

**Derived decision:** Kepler's observed request-history-dependent one-second waits are unsatisfactory for responsive editing. It is no longer the preferred experimental transport. 083's spacing/quiet-pause controls support an internal pacing hypothesis but do not identify a fixed quota or prove that HTTP itself is unsuitable.

[Own DLL/Named-Pipe bridge and logger](NATIVE_BRIDGE.md) now passed direct strings under GC pressure, logging without ACSEDebug, sandbox-to-external-Python communication and 1,000 acknowledged messages each way between two local clients. ACSE remains the loader. This is promising local transport and original diagnostic code, not a complete interactive debugger or released remote netcode.

Step 12 will port one ordinary manual Create plus duplicate checks to this path; no result is reviewed yet. Earlier HTTP scenery/recovery findings remain valid in their original scope and are not automatically transferred. Companion remote transport, authentication, pipe reconnect, durable recovery and the reported exit crash remain open.

The remaining sections preserve the tested historical HTTP contract.

## Verified historical HTTP behavior

The transport blocker recorded at diagnostic 034 has been resolved for the tested **native HTTP GET to an owner-controlled loopback server**. Diagnostics 037b–047 demonstrate text request/response transport. The original 038 demonstrated sequential A/B/A-again runs; a later fresh-session tropical-park repeat passed on two simultaneous local clients. 040–042 connect downloaded commands to real scenery creation/movement with session-local duplicate suppression.

Test environment: Windows / Steam, PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0, ACSEDebug 0.014. Binary hashes are not established in this public record. Claims apply to this tested setup, not every PC2 release.

## Current two-client result and latency boundary

071–078 passed scripted scenery operations, duplicate/native-history checks and controlled ACK/TCP/server/checkpoint-client recovery on two running local clients. Corrected 079–081 captured one ordinary host-confirmed Create and replicated it once to the guest while keeping the next preview local (080/081). General manual Move/Delete and remote play remain open.

Completed 082: all 14 phases, 1,200 correct probes, native task joins complete, both clients/Python/server Completed=true. B starts fast but later native calls take roughly 940 ms even alone, with backlog 128 and on a separate port. Server accept-to-reply and write sections stay fast. Same-sandbox Python has total medians 0.94–0.99 ms and max 24.04 ms. These observations narrow the suspect path but do not prove a fixed rate limit, exact Kepler cause or TCP failure. [Full evidence and method](HTTP_LATENCY_080_082.md).

No active Frontier connection is required for the tested local ready context. That does not imply the native request scheduler is unlimited, a future service shutdown is irrelevant, or remote HTTPS/security already works.

## Runtime-confirmed native request contract

The original diagnostic component's request worker (037b–047; retained 047 source, worker around lines 410–489) uses:

```lua
-- Fragment INSIDE an api.task.Spawn worker, after context readiness.
local request = api.online.CreateKeplerHttpRequest(playerID)
request:SetUrl(loopbackURL)
request:SetMethod(api.online.KeplerMethod_Get)
request:SetGenerateAuthCode(false)
local success, resultCode, response = api.online.RunKeplerRequest(request)
-- Success/error code is separate from the HTTP status.
-- For the tested successful responses:
-- success == true
-- response.nResponseCode == 200
-- response.sBody is the complete response text string
```

This is a contract illustration, not a standalone script: retain the request AND task until completion, poll `api.task.JoinAsync(task)` from Advance, handle the result, then release references. The tested implementation has **one in-flight native request**, not a background thread manipulating the editor. Dispatch validated editor actions through the world component's controlled Advance/coroutine sequence. Stop on world/owner changes. A pending request is not assumed cancellable just because a local timeout elapsed.

The constructor receives the game-owner/player ID; the destination is set with **SetUrl**, not passed as a guessed constructor argument. Generic HTTP body access is the **third RunKeplerRequest return's sBody**, not a second-return string or a specialized request getter. GET, plain loopback HTTP and text downloads are runtime-tested. Remote HTTPS, certificate/proxy behavior, uploads/POST/PUT, arbitrary binary bodies, authentication headers, compression, connection reuse and long-poll timeout behavior are **not established by these tests**.

## Readiness and Frontier dependence

Diagnostic 045's gate, also used by 046/047:

1. `api.player.GetGameOwner()` identifies a valid player according to `api.player.IsValid(playerID)`.
2. `api.online.IsKeplerAvailable()` is true.
3. `api.online.IsKeplerUser(playerID)` is true.

Observe FrontierAccountManager `IsConnected` / `IsConnecting` for evidence, but **do not require IsConnected for a local request**. The verified gate works without an active Frontier connection.

- 037: early invocation failed, Kepler result **19 (Cancelled)**, no request at the local server. The enum interpretation comes from prior private native inspection; cancellation alone does not identify its cause.
- 037b: waiting for full online initialization allowed a successful request, but was an unnecessarily strong gate.
- 043: after a previously established Frontier connection was lost, three LOSS-group local requests still succeeded. The recovery part did not finish; the overall test was not a completed recovery proof.
- 044: internet disconnected before launch; an early no-context request failed, then two succeeded after a Kepler user context appeared, without Frontier login. This is NOT proof of success without any Kepler context.
- 045: all three requests succeeded with Kepler context and Frontier disconnected.

**Conclusion within scope:** active Frontier connectivity is not required once the tested native context is ready, including the tested offline startup. Complete independence from cached account state, Steam availability, licensing, a future SDK change or permanent service shutdown remains unknown. Remote peers still need a working network path to their relay.

## Why the Gameface/JavaScript route was set aside

035b established a custom Lua-to-JavaScript-to-Lua message roundtrip and view/resource cleanup. 036/036b JavaScript HTTP attempts reported 404 while the owner-controlled server saw no corresponding request. A separate Python control from the same PC2CoopHost sandbox reached exactly that server with HTTP 200 and the expected marker/body.

Prior private inspection of PC2's native Gameface resource handling reported resource-oriented dispatch rather than demonstrated forwarding of these external requests. The controls are consistent with a rejection/404 in the embedded resource path, not a missing Python route or a general Sandboxie loopback block. Do not extrapolate this to every Gameface configuration or claim the control proves all per-process network permissions. At that stage, the demonstrated HTTP path was native Kepler rather than JavaScript. The newer preferred local boundary is the own DLL/pipe channel described above.

## Capability history (031–034)

031 found no usable standard io/os or LuaSocket; the file roundtrip was skipped, not passed. package.loadlib existed but was not invoked. Raw API enumeration in 032 was incomplete. In 033/034, five candidate named fields resolved as functions while nine invented fields were nil. These were presence tests only; later request diagnostics establish behavior.

Named controls: api.http.URLEncode, api.online.CreateKeplerHttpRequest, api.online.GetKeplerEndpoint, api.online.RunKeplerRequest and api.IRC.RemoveIRCNotificationListener. No custom IRC sender/channel was established; no need to instantiate account/workshop managers for our relay.

## Observed in supplied extracted material

These are source anchors, not redistributed source. File line numbers refer to the supplied reconstructed versions and may change on re-decompilation.

| File / function | Observed call path | Meaning |
| --- | --- | --- |
| managers.webrequestmanager_unluac.lua, RunRequest, around line 69 | Dispatches named methods on api.franchise with a callback and supplied arguments | A specialized dispatcher, not a demonstrated arbitrary-URL HTTP client |
| managers.workshopmanager_unluac.lua, _RunCreatorListRequest, lines 601–622 | CreateKeplerProfilePagedListRequest(player), setters, RunKeplerRequest(handle), handle:GetResults() | Typed profile result |
| Same file, OnFollowCreator, lines 2223–2225 and unfollow around 2259–2262 | CreateKeplerProfileFollowRequest(player), SetProfileID, RunKeplerRequest | Specialized account mutation; do not use for transport probing |
| Same file, _RunFollowedUsersIDRequest, lines 2417–2424 | RunKeplerRequest(handle), handle:GetIDs() | Typed ID list |
| managers.newsfeedmanager_unluac.lua, _SpawnRequestNewsfeedTask / _RequestNewsfeedData, lines 200–230 | CreateKeplerNewsfeedRequest(player), settings, RunKeplerRequest, GetArticles; separate image request | Specialized content retrieval |
| Same file, around line 163 | article:GetBody() | Article text, not a demonstrated raw HTTP-response accessor |
| managers.frontieraccountmanager.lua.bin, _RunKeplerRequest, original source line 1772 | Calls api.online.RunKeplerRequest(request) and reads success/result code | Corroborated by bytecode; no arbitrary URL supplied |
| FrontierAccountManager, SearchPlayers / _DownloadImage | URLEncode and specialized franchise SearchPlayers / DownloadImage | URL encoding and image download are not proof of generic text HTTP |
| online park-session / notification managers, PrepareIRCListeners | Structured IRC notifications for park locks, friendships and updates | No custom sender, server address or general IRC protocol access demonstrated |

In these specialized source call sites, RunKeplerRequest receives a request handle and callers capture two returns. Later generic HTTP runtime tests capture a THIRD return containing the response table. Neither the second return nor a specialized article getter is the raw HTTP body. Generic HTTP uses response.sBody as documented above.

Some online-notification handlers acknowledge/delete stored notifications. Instantiating such a manager is not a passive read-only probe.


## Decompilation qualification

The supplied FrontierAccountManager bytecode uses PC2's custom Lua 5.3 header. A separate analysis copy was normalized (format byte and omitted instruction-size byte), retaining instruction bytes. Parsing consumed all 96,058 normalized bytes and identified 135 prototypes including the main block; no vector constants occurred.

unluac v1.2.3.569 failed in the debug-informed decompile at source lines 291–341 (PrepareToShutdown). --nodebug produced output through file end with 134 function declarations plus main. A complete disassembly was also produced privately. Descriptive top-level aliases were added only to a separate reading copy; local names remain partly synthetic. Reconstruction is not the original source and was not executed or shown runtime-equivalent.

No complete decompiled scripts, bytecode or game resources are distributed in this repository.


## Next acceptance gate

Simultaneous local messaging/scripted scenery and one manual host Create have now passed. 083 has tested spacing/history and quiet recovery; 084 has passed the own local transport through Step 11. Next validate Step 12 native-pipe Create, recovery/lifecycle, manual Move/Delete and a secured remote companion route. An internet relay package was prepared but no remote friend test has passed yet. Read the [proposed protocol and failure rules](HTTP_SYNC_DESIGN.md); these rules are design work, not an already implemented co-op system.

## Local recovery update — diagnostics 055–063

HTTP DELETE and live-session native Undo/Redo passed with duplicate/order/ID-conflict guards. 059 deliberately refused first ACK storage with 503; 060/061 reset TCP before first ACK storage. Successful repeats retained native task/request references until completion and did not repeat editor actions. All 38 HTTP tasks joined in 061, including expected failures.

The tested reset response was `nativeSuccess=false`, Kepler result 3, reported status field 2, nil body. The value 2 is not an HTTP status. Treat the complete tested tuple as failure evidence; do not reinterpret every non-200 response as a TCP reset or discard still-running tasks on timeout.

062 resumed a journal-backed server after an actual process exit while the client stayed alive. 063 resumed a restarted client from a completed save/manifest checkpoint. These extend local recovery evidence but do not prove arbitrary crash safety or simultaneous-client convergence. [Exact outcomes and limits](RECOVERY_AND_HISTORY.md).

## Historical HTTP alternatives (not validated fixes for Kepler)

Long polling could remove the explicit idle-poll pause; persistent HTTP could reduce connection setup; a sidecar could decouple remote networking from the game. None has resolved the 082 behavior in a runtime test. Retain one owned request/task until completion, use bounded queues and do not assume a local timer cancels native work. A suspected request limit must be measured and respected, not treated as a confirmed implementation detail or bypass target.
