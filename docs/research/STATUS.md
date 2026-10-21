# Current research status

**Updated 2026-10-21 through diagnostic 034.** [Overview](../../README.md) · [Runtime record](../testing/RUNTIME_TESTS.md) · [Transport](../networking/TRANSPORT.md)

## Verified in a running game

Reported environment: Steam / Windows, PC2 **1.11.4.1011357**, ACSE **0.729**, ForgeUtils **2.7.0**, ACSEDebug **0.014**. Executable/package hashes are not established in this public record.

- The custom ACSE component registers, activates in a park and is recreated across world transitions. Availability of ForgeUtils AddHook is established; execution of a particular hook is not.
- Diagnostics **007–018** progressed from real undo-hierarchy creation to creation of base-game scenery resource **AQ_Rock_04**, editing its transform, completed preview/cancel, and a single completed commit. The created world part was checked over two component Advances.
- **019** saved a new local park and, in a separate verification-only run after manual loading, found part **1182**, AQ_Rock_04, at **(10, 5, 10)**.
- **020–022** edited the existing part, tested restoration, committed movement to **(15, 5, 10)**, and verified that movement after a new local save and manual reload.
- **023–025** tested deletion preview/cancel, committed deletion and verified continued absence of the part and entity mappings after save/reload.
- **026–027** distinguished two identical resources by mod-owned keys and local part IDs; moving A and deleting B left the other object unchanged.
- **028** processed eight scripted inputs, skipped four duplicate commands and committed four actions.
- **029** rejected four ordering errors and four same-ID/different-payload conflicts; four valid actions committed, A ended at (15, 5, 10), B absent.
- **030** produced matching canonical states after all twelve inputs in separate A/B-labelled runs: **120/120 STATE records matched** after removing instance labels and timestamps. This is repeatability, not live client replication.
- **031–034** investigated transport capability. Ordinary field lookups resolve five candidate API functions; nine invented names all return nil. No candidate network function was invoked by 033/034.

Details and evidence boundaries: [diagnostics 007–034](DIAGNOSTICS_007_034.md).

## Current blocker: command transport

The tested Lua environment exposes neither standard io/os nor an available LuaSocket module. The planned file roundtrip was **skipped**, not passed. Native HTTP/Kepler functions are promising candidates, but arbitrary URLs and raw response text remain **unknown**.

Static source shows RunKeplerRequest(requestHandle) used with specialized requests. No CreateKeplerHttpRequest call site has been found in the supplied corpus, including the now fully decompiled FrontierAccountManager. Do not guess a native constructor signature. See [transport investigation](../networking/TRANSPORT.md).

## Not yet demonstrated

- Two simultaneously running independent clients exchanging commands; remote gameplay or playable co-op.
- Capture of ordinary player-confirmed scenery edits. The diagnostics use scripted commands.
- Authority/ownership, concurrent edits, durable deduplication, reconnect, crash recovery, late join or resynchronization.
- Persistent mod-owned network-ID mapping across saves and different clients.
- General object support beyond the tested rock; terrain, paths, tracks, rides and simulation synchronization.
- Undo/redo behavior after committed multiplayer operations.

## Test context and historical qualifications

005/006 were explicitly reported as copied-install runs **outside Sandboxie**. A separate host Sandboxie configuration reached the menu with UnstrippedToken=y. Later tests were performed by the owner using the test-copy workflow; do not retroactively infer process isolation from a log filename or relocated path. A second Sandboxie guest/client and complete Steam Cloud isolation remain unvalidated.

UnstrippedToken reduces token isolation; its startup success does not identify Denuvo as the cause. The initial world-transition crash was not experimentally attributed solely to missing Shutdown(). ACSEDebug logging works; its cross-game GUI/native features remain unvalidated.

## Next milestone

Find a supported request constructor, own-server destination and readable response path, or establish an original local bridge. First transport test should exchange a harmless identifier/acknowledgement, with **no game edits**. Only after that connect the existing command checks to two genuine clients. Keep raw logs, saves, binaries and complete game decompilations private.
