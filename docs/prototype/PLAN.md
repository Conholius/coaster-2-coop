# Scenery co-op prototype: completed gates and next work

**Updated 2026-10-24 through diagnostic 084 Step 11; Step 12 not yet verified.** [Status](../research/STATUS.md) · [Latest ledger](../research/DIAGNOSTICS_069_082.md) · [Protocol](../networking/HTTP_SYNC_DESIGN.md) · [Latency](../networking/HTTP_LATENCY_080_082.md)

## Initial goal achieved within diagnostic scope

Two local PC2 clients can apply scripted AQ_Rock_04 Create/Move/Delete/native Undo/Redo events through a loopback relay, suppress duplicate effects and pass controlled reconnect/restart tests. One player-confirmed host placement replicates to the guest while the host's next unconfirmed preview remains active and local.

This is an owner-run diagnostic prototype, not a finished multiplayer mod or automated QA suite. Local Part-IDs and Entity-IDs are not universal identities. No internet deployment or general manual Move/Delete capture has passed.

## Own transport milestone

The preferred next path is own DLL -> bounded local Named Pipes -> external companion. Direct Lua strings under GC, own logging without ACSEDebug and 1,000 verified messages each way between two local clients passed. Kepler was set aside because the measured history-dependent seconds-long waits are unsatisfactory, not because HTTP itself cannot carry operations. [Evidence and timings](../networking/NATIVE_BRIDGE.md).

The owner still reports a game-exit crash reporter; successful communication completion is not a shutdown-safety proof. Own diagnostic output is implemented; a full interactive debugger is not.

## Next implementation gates

1. Execute 084 Step 12: transfer one ordinary host Create through the new native path, preserve its next preview, commit once on Guest, verify mappings/transform and reject an identical duplicate. Code is prepared; no reviewed outcome.
2. Capture manual host Move/Delete and repeated Create safely after actual confirmation; distinguish preview updates/cancellation from durable edits.
3. Build generalized logical identity/revision/status management and origin/remote echo suppression, then guest intent acceptance and conflict ownership.
4. Preserve native completion/verification and bounded work; benchmark request count, application latency and Advance gaps separately.
5. Validate a secured external-companion remote route with a friend, including packaging, auth, failures and bounded reconnect. First test local pipe/companion interruption and world-deactivation behavior.
6. Integrate registry metadata/checkpoints and test ambiguous client-crash windows, mismatched saves, replay and late join.
7. Prototype host/join UI after identifying actual PC2 runtime extension points. The product direction is a shared host-controlled session and save workflow; independently loaded local test parks are a testing scaffold.
8. Expand resources, full transform validation, groups/dependencies and Undo ownership, then soak and presented-frame tests.

## Reproducibility and acceptance

Keep matched protocol sessions, canonical payloads, local mappings, actual native mutation counts, application proofs, task joins and explicit expected/observed results. Retain raw evidence privately and publish summarized findings. Corrected successful runs must be distinguished from initial/aborted attempts. A final summary on failure is not Completed=true.

For 082 reproduce: both game clients in tropical parks, separate control listener, ten warmups plus 50 measured probes per active role/configuration, seven configurations then reversed order, fixed bodies and request IDs, 50 ms requested spacing, Python control in Guest sandbox. Leave all participants alive until all fourteen phase barriers and final completions finish.

## Boundaries

A guest-local checkpoint save was specifically permitted in 078, not an established general guest-save policy. Native Undo stacks do not survive the tested reload; replay safety and optional cross-session Undo are separate concerns. A menu/lobby does not implement world transfer. No full guest/staff/economy/terrain/path/track/ride simulation, production security or released playable co-op is claimed.
