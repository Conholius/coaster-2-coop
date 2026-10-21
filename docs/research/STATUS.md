# Current research status

**Updated 2026-10-21.** [Project overview](../../README.md) · [Runtime tests](../testing/RUNTIME_TESTS.md) · [Open questions](OPEN_QUESTIONS.md)

## Verified in a running PC2 copy

Test environment reported in logs: **Planet Coaster 2 1.11.4.1011357**, **ACSE 0.729**, **ForgeUtils 2.7.0**, **ACSEDebug 0.014**. The 005/006 sessions used a **copied test installation outside Sandboxie**.

- `PC2CoopDiagnostics` loads from `Main.ovl`; `LuaDB.Init()` logs, and `require("forgeutils.hookmanager")` returns a table exposing `AddHook`. **This is not proof a ForgeUtils hook was executed.**
- After fixing the packaged `Main.ovl`, diagnostic **005** confirmed component registration and menu-world `Configure`, `Init`, `OnWorldActivation`.
- The minimal component **with `Shutdown()`** ran `OnWorldDeactivation` and `Shutdown` when switching worlds, was recreated for `tropicalworld`, and let the park load and exit normally.
- Diagnostic **006** inspected `tropicalworld`: `GetWorldAPIs()` succeeded, `worldAPIs.editors` was accessible, and `CreateUndoOperationsHierarchy`, `api.undo.PreviewChangeList`, `CanCommitPreview`, `CommitPreview`, and `CancelPreview` were present as Lua-callable functions. **No editor method was executed beyond obtaining the world APIs and inspecting fields.**
- ACSEDebug produced useful trace logs. **Its cross-game debug GUI / native functions remain unverified and should not be used.**

Reproducible timeline and caveats: [runtime diagnostics](../testing/RUNTIME_TESTS.md).

## Static research (not yet runtime-validated for scenery creation)

- Decompiled PC2 scenery code in `SubModePathSnapPlacement` and brush placement shows an undo hierarchy, `CreateNewPart`, edit/clone cases, a preview lifecycle and delayed commit/cancel. Decompiled code has known vector-placeholder caveats.
- PC-CC's existing PC2 source shows an editor hierarchy followed by a full change list and preview/commit; its demonstrated operation is **track-specific**, not proof of scenery placement from our mod.
- Game save infrastructure delegates to world serialization; `LocalChangeManager` tracks save versions, not editor actions.
- `InstantiatePrefab` creates game entities but has **not** been shown to create an editable, persisted park scenery placement.

See [placement](../placement/README.md), [public source investigation](../placement/ACSE_FORGEUTILS.md) and [evidence index](EVIDENCE.md).

## Historical failure — no longer the current blocker

An earlier custom component **without a `Shutdown()` method and with extra editor probing** crashed when leaving `startscreenglobe` for a new tropical park. Disabling its registration was a successful A/B control. A later **minimal component with `Shutdown()`** made the transition successfully, followed by successful editor *presence* checks in diagnostic 006. Because the modifications were not isolated one at a time, **the exact earlier crash cause is still unknown**. Do not present missing `Shutdown()` as the proven cause. See [research log](LOG.md).

## Next milestone — not yet run

**Diagnostic 007:** call `worldAPIs.editors:CreateUndoOperationsHierarchy()` in `tropicalworld`; inspect the returned hierarchy without adding edits, previewing or committing. **No successful hierarchy creation has yet been logged.**

After that, establish exact scenery-operation parameters, run a controlled single-part preview/commit on a disposable park, verify save/reload persistence and only then consider two-client replay.

## Sandboxie and test safety

A **separate, previously configured** Sandboxie host/guest setup is recorded in [ISOLATED-INSTALLATION.md](../ISOLATED-INSTALLATION.md). Its `UnstrippedToken=y` compatibility setting weakens token isolation; the relation to Denuvo is **unproven**. **Do not conflate this configured setup with the non-Sandboxie 005/006 runtime sessions.** The guest has **not** been validated as a second PC2 client.

Use backups and disposable saves. No production game files, logs, credentials or dumps are published here.

## Still unproven

- Invoking `CreateUndoOperationsHierarchy` in our component and inspecting its returned hierarchy.
- Programmatically creating, committing, saving and reloading an ordinary scenery part.
- Detecting only user-confirmed edits, mapping stable placement IDs across clients, network transport, conflict recovery and real-time co-op.
