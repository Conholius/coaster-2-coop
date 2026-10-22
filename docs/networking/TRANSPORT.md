# Native HTTP transport in Planet Coaster 2

**Updated 2026-10-22; runtime evidence through diagnostic 049. Diagnostic 050 is prepared / owner testing; no result is claimed.**

[Current status](../research/STATUS.md) · [035–049 evidence ledger](../research/DIAGNOSTICS_035_049.md) · [Proposed synchronization](HTTP_SYNC_DESIGN.md) · [Measurements](BENCHMARKS.md)

## Verified in a running game

The transport blocker recorded at diagnostic 034 has been resolved for the tested **native HTTP GET to an owner-controlled loopback server**. Diagnostics 037b–047 demonstrate text request/response transport. 038 demonstrates a server retaining messages between **sequential A, B and A-again game runs**, not simultaneous clients. 040–042 connect downloaded commands to real scenery creation/movement with session-local duplicate suppression.

Test environment: Windows / Steam, PC2 1.11.4.1011357, ACSE 0.729, ForgeUtils 2.7.0, ACSEDebug 0.014. Binary hashes are not established in this public record. Claims apply to this tested setup, not every PC2 release.

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

Prior private inspection of PC2's native Gameface resource handling reported resource-oriented dispatch rather than demonstrated forwarding of these external requests. The controls are consistent with a rejection/404 in the embedded resource path, not a missing Python route or a general Sandboxie loopback block. Do not extrapolate this to every Gameface configuration or claim the control proves all per-process network permissions. The supported path for our tested transport is native Kepler HTTP; JavaScript is not required.

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

Use two independently authorized real clients at the same time, with matching park baselines, first for harmless messages and then a scripted scenery command. An internet relay package was prepared but no remote friend test has passed yet. Read the [proposed protocol and failure rules](HTTP_SYNC_DESIGN.md); these rules are design work, not an already implemented co-op system.
