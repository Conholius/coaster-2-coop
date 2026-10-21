# Runtime tests — PC2 lifecycle, scenery and transport capability

[← Documentation guide](../README.md) · [Current status](../research/STATUS.md) · [Scenery research](../placement/README.md)

**As of 2026-10-21.** This page separates experiments that **ran in PC2**, tests that were **not run**, and historical failures. Log excerpts here are short manually transcribed observations; **complete private logs are not committed**.

## Environment and isolation boundary

| Property | Observed or reported value |
| --- | --- |
| PC2 game build | `1.11.4.1011357` (self-reported at log startup) |
| ACSE | `0.729` |
| ForgeUtils | `2.7.0` |
| ACSEDebug | `0.014`, cross-game package; logging works, GUI safety not established |
| Diagnostic mod | `PC2CoopDiagnostics`, Lua code packed as `Main.ovl` |
| Game execution for diagnostics 005 and 006 | **Separate copied test installation, but *not* launched through Sandboxie**, per tester confirmation |
| Game / mod binary SHA-256 | Not published / not established in these records |

The existence of an already configured Sandboxie `PC2CoopHost` environment is a **separate** setup fact. Do not claim the successful 005/006 game sessions were sandboxed. See [isolation setup](../ISOLATED-INSTALLATION.md).

## Chronological test record

| Test / private log | What changed | What was actually observed | Interpretation |
| --- | --- | --- | --- |
| Initial component test — `Planet Coaster 2(1).log` | Enabled custom `AddLuaComponents` diagnostic with editor availability probes and **no explicit `Shutdown()`** | Component initialized and menu-world method checks were true; park transition stopped after old-world shutdown | **Failure.** Underlying cause unknown; no native exception trace. |
| A/B control — `Planet Coaster 2(2).log` | Disabled **only** custom component registration while retaining ACSEDebug and `LuaDB.Init()` | Tropical park loaded, ran over 60 seconds and shut down normally | Component involvement strongly suggested; does **not** prove `Shutdown()` alone fixed it. |
| Packaging controls — `Planet Coaster 2(3).log`, `(4).log` | Intended minimal world-component test, but earlier `Main.ovl` still in use | `LuaDB.Init()` logged but custom component was **not registered** | **Not** valid evidence for the new component. |
| **Diagnostic 005 — `Planet Coaster 2(5).log`** | Added `VERSION TEST 005`; rebuilt/copied the correct `Main.ovl`; registered minimal component with `Shutdown()` | New version marker, registration, `Configure`, `Init`, and `OnWorldActivation` appeared in menu world | Confirmed packaging and basic component lifecycle. |
| **Lifecycle / park transition — `Planet Coaster 2(6).log`** | Continued with the registered minimal component | Menu-world `OnWorldDeactivation` and `Shutdown`; new component registered/configured/initialized in park; `tropicalworld` activated; park completed load and normal exit | **Successful** park-world component lifecycle. Tester also changed screen mode while running. |
| **Diagnostic 006 — `Planet Coaster 2(7).log`** | Added guarded, presence-only checks on world activation | In `tropicalworld`: `GetWorldAPIs` succeeded; `editors` accessible; `CreateUndoOperationsHierarchy` and four `api.undo` methods present as functions. Park finished loading and later deactivated normally. | **Successful API presence test**, not a method-invocation or scenery-edit test. |
| **Diagnostic 007** | Invoked hierarchy construction in the park | Returned hierarchy used successfully in subsequent tests | Passed; historical planned status superseded. |

### Key runtime checkpoint: diagnostic 006

In the active `tropicalworld`, the log reports:

```text
[PC2-COOP] Current world: tropicalworld
[PC2-COOP] GetWorldAPIs succeeded: true
[PC2-COOP] Editors API accessible: true
[PC2-COOP] CreateUndoOperationsHierarchy available: true
[PC2-COOP] Undo API accessible: true
[PC2-COOP] PreviewChangeList available: true
[PC2-COOP] CanCommitPreview available: true
[PC2-COOP] CommitPreview available: true
[PC2-COOP] CancelPreview available: true
[PC2-COOP] EDITOR DIAGNOSTICS 006 COMPLETE
```

These entries are **presence checks only**. `pcall()` around a Lua call is not a guarantee against a native engine crash.

## Diagnostics 007–034: later runtime results

The detailed [diagnostic ledger](../research/DIAGNOSTICS_007_034.md) records each step and its limits. These owner-run tests are not a two-client automated QA suite.

| Range | Result |
| --- | --- |
| 007–013 | Hierarchy/change list, CreateNewPart, StartEditingPart_Scenery, transform reads and reversible cross-Advance movement |
| 014–018 | Completed preview, cancellation, committability, one commit and world-instance verification |
| 019–022 | Creation and existing-part movement persisted through new local saves and manual verify-only reloads |
| 023–025 | Deletion preview restored on cancellation; committed deletion persisted after save/reload |
| 026–027 | Two equal-resource rocks addressed independently by own keys/local IDs |
| 028 | Eight commands, four duplicate skips, four commits |
| 029 | Four order errors and four ID conflicts rejected; four valid commits |
| 030 | A/B separate runs: 120 matching canonical STATE records, covering all twelve inputs |
| 031 | io/os/LuaSocket unavailable; file/socket tests skipped; loadlib present but untested |
| 032–034 | Raw enumeration incomplete; five normal-access controls function; nine invented names nil |

### Retained evidence rechecked during this update

- 022 Phase 1: CommitPreview call count 1, operation completed, save returned success; Phase 2 checks the loaded part/resource/position without mutation.
- 025 Phase 1: one completed deletion commit and successful save; Phase 2 checks absent part and mappings.
- 030 A/B: twelve verified snapshots, each ending with four applied and eight rejected inputs; canonical comparison excludes instance labels/timestamps and local IDs.
- 034: checked=14, controlFunctions=5/5, inventedFunctions=0/9, inventedNil=9, failedOrSkipped=0, Completed=true.

No tests were rerun in the game for this documentation update. Earlier results are consolidated from prior private log reviews; retained selected logs were reread.

## Next experiment: transport without game edits

Establish exact HTTP constructor and response semantics before a native request. The initial transport test should send a harmless message to an owner-controlled endpoint and read an acknowledgement, with no editor, commit or save calls. This test has **not run**. [Investigation](../networking/TRANSPORT.md).

## Not demonstrated

- Simultaneous independent clients, message transport or live co-op.
- Ordinary user-edit capture, general undo/redo, crash/reconnect recovery or persistent deduplication.
- Complete Steam/Cloud isolation, a working second sandboxed client, or a Denuvo root cause.
- Applicability beyond the tested base-game rock and local save workflow.

## Evidence handling

Keep unredacted logs, absolute Windows user paths, game executable hashes, game binaries, mod OVL files and saves **locally**, not in this public repository. See the [evidence index](../research/EVIDENCE.md) and [chronological research log](../research/LOG.md).
