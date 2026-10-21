# Command transport investigation

**As of 2026-10-21; diagnostics 031–034.** [Current status](../research/STATUS.md)

## Verified runtime capability

| Test | Observation | Boundary |
| --- | --- | --- |
| 031 | Lua 5.3; global io and os nil; require("io"), require("socket") and require("socket.core") failed | File write/read/cleanup and socket-object tests skipped; no transport established |
| 031 | package.loadlib present as function | Not invoked; native ABI/module loading untested |
| 032 | Raw table enumeration found only part of the API; protected metatables returned strings | Empty raw table is not proof of absent API methods |
| 033 | Ordinary protected field accesses returned function for the five names below | No returned function invoked |
| 034 | Same five controls returned function; all nine invented names returned nil; checked=14, failedOrSkipped=0, Completed=true | Rejects the tested arbitrary-name-wrapper hypothesis, not a signature or functionality test |

Five resolved names:
- api.http.URLEncode
- api.online.CreateKeplerHttpRequest
- api.online.GetKeplerEndpoint
- api.online.RunKeplerRequest
- api.IRC.RemoveIRCNotificationListener

034 used three invented names (PC2Coop034_NoSuchFunction_7F29A1, _B84C62 and _D903E5) in each of http, online and IRC. All returned nil. Neither listeners nor network requests were started by 033/034.

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

RunKeplerRequest receives one request handle in the observed calls, with two captured returns (success and result/error). Payloads are read through specialized handle getters; the second return is not established as raw response text. Callers use task/coroutine contexts; do not infer safe synchronous behavior in Advance.

Some online-notification handlers acknowledge/delete stored notifications. Instantiating such a manager is not a passive read-only probe.

## CreateKeplerHttpRequest: still unknown

No call site was found in the currently supplied Decompiled corpus, including binary string searches and the complete FrontierAccountManager reconstruction. This is a corpus limit, not proof the function is unusable.

An older PC2_1.0.6.429191_API.lua lists the constructor and KeplerMethod_Get/Put/Post/Delete constants. That dump does not supply a signature or prove current constant values. The current function value was separately verified by 033/034.

Unanswered:
- Constructor arguments and ownership/lifetime.
- Free URL versus relative/predefined Frontier endpoint.
- HTTP methods, headers and body setters.
- Response bytes/text, status, cancellation and completion behavior.
- Loopback/plain HTTP availability, authentication requirements and sandbox access.

## Decompilation qualification

The supplied FrontierAccountManager bytecode uses PC2's custom Lua 5.3 header. A separate analysis copy was normalized (format byte and omitted instruction-size byte), retaining instruction bytes. Parsing consumed all 96,058 normalized bytes and identified 135 prototypes including the main block; no vector constants occurred.

unluac v1.2.3.569 failed in the debug-informed decompile at source lines 291–341 (PrepareToShutdown). --nodebug produced output through file end with 134 function declarations plus main. A complete disassembly was also produced privately. Descriptive top-level aliases were added only to a separate reading copy; local names remain partly synthetic. Reconstruction is not the original source and was not executed or shown runtime-equivalent.

No complete decompiled scripts, bytecode or game resources are distributed in this repository.

## Next test design (not run)

Find a real constructor call/type description before invoking native methods. If unavailable, investigate an original external bridge and its Lua ABI separately. First prove message/ack transport with no editor/save calls. Then connect a validated, ordered, session-deduplicated command to two actual clients and compare canonical states. A remote connection does not itself establish safe replay or durable exactly-once effects.
