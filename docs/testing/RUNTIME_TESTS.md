# Runtime tests — PC2 world lifecycle and editor interface

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
| **Diagnostic 007** | Proposed `editors:CreateUndoOperationsHierarchy()` plus result/type/method inspection | **Not run; no log supplied.** | Next planned experiment; do not mark as passing. |

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

## Next experiment: diagnostic 007

**Goal:** from our own ACSE world component in `tropicalworld`, call:

```lua
local hierarchy = editors:CreateUndoOperationsHierarchy()
```

**Only after confirming the expected world and editor interface:** record whether the call succeeded, the returned value/type, and the **presence** of hierarchy methods such as `GetFullUndoChangeList`, `CreateNewPart`, `CreateNewParts`, `EditExistingPart`, `CloneExistingPart`. Do **not** call creation/editing methods, begin a preview, or commit.

The method-call pattern appears in [PC-CC's PC2 importer source](https://github.com/parker02311/PC-CC/blob/e722a387d407e532d0c6be914a97b7a0121f21b7/Mod_Importer/Main/Managers/Mod_Importer.UIManager.lua), but compatibility of this call with **our** component is still **unverified**.

**Suggested procedure:**

1. Back up the working diagnostic 006 `Main.ovl` and the disposable test park.
2. Verify which **copied install** and which **sandbox / Steam context** will run. Do not substitute an older launcher that restores baseline mods without inspecting it; review [Sandboxie notes](../ISOLATED-INSTALLATION.md).
3. Rebuild from the intended `Main` source using Cobra Tools, with `Flatten Lua Subfolders`, and copy the new `Main.ovl` to the diagnostic mod in the copied installation.
4. Start PC2; verify the version marker and custom component registration in the fresh log.
5. Load a **new empty test park**, wait approximately 60 seconds, exit normally and inspect the log.
6. If stable, record results and only then plan `GetFullUndoChangeList()` and controlled scenery operations.

**Pass condition:** hierarchy creation reported success in `tropicalworld` **and** the game safely transitions and exits. **Failure condition:** error, nil hierarchy, hang or crash; retain the relevant private log and roll back to diagnostic 006.

## Not demonstrated

- No runtime mod call creating scenery, `PreviewChangeList()`, `CommitPreview()`, or scenery operation token flow.
- No confirmation that a created object is selectable/editable as a normal park part.
- No save/reload persistence, no second independent client, no replication and no playable co-op.
- No proof that Sandboxie isolates all Steam/Cloud/user-profile activity or that Denuvo caused the previous sandbox startup failure.

## Evidence handling

Keep unredacted logs, absolute Windows user paths, game executable hashes, game binaries, mod OVL files and saves **locally**, not in this public repository. See the [evidence index](../research/EVIDENCE.md) and [chronological research log](../research/LOG.md).
