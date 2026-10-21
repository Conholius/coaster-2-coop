# Open questions and next experiments

[← Current status](STATUS.md) · [Runtime tests](../testing/RUNTIME_TESTS.md) · [Prototype plan](../prototype/PLAN.md)

**Prioritized as of 2026-10-21.** A method that **exists** in a Lua API has not necessarily been **called**, and a successful call is not proof of persistence.

## Immediate blockers

1. **Diagnostic 007 — hierarchy invocation:** Can our ACSE component in `tropicalworld` execute `editors:CreateUndoOperationsHierarchy()`, receive a valid hierarchy and inspect its methods without destabilizing a world transition? **Not tested.**
2. **Exact scenery edit arguments:** Which base-game scenery resource and parameters does `CreateNewPart` require, how are completion tokens awaited, and which coordinate/ID types does the editor use? Static call sites are known, not yet confirmed by a runtime edit trace.
3. **Durable edit:** Can a single, user-authorized scenery change go through preview, commit, save, exit and reload without duplicates? **No evidence yet.**
4. **Confirmed-change capture:** Where can we observe *committed* edits rather than transient previews or gizmo movement?
5. **Identity mapping:** How do `EntityID`, `PlacementID`, `PlacementPartID`, saved IDs and mod-defined network IDs relate across reload and clients?

## Next phase after persistence

6. **World serialization:** Which implementation backs the park world's `worldserialisation` API, and how can we compare lasting placement state?
7. **Two-client baseline:** Can two separately authorized clients load identical base parks and compare canonical inventories while keeping Steam/profile and save boundaries isolated?
8. **Replication protocol:** Which ordering, validation, authority, retry and rollback model reproduces the same scene without conflicts or corruption?
9. **Version and distribution:** Which exact game/ACSE build hashes and licensing or launcher constraints apply to a reproducible package?

## Environment and safety uncertainties

- **Sandboxie:** A separate `PC2CoopHost` setup has previously reached the menu with `UnstrippedToken=y`, but the **successful 005/006 editor sessions were run outside Sandboxie**. Can the same diagnostic and later controlled edits be reproduced in the sandbox? Does the guest launch independently? **Unknown.**
- **Earlier world-transition crash:** Minimal component with `Shutdown()` and removed probes works; was the original failure caused by the missing callback, unsafe probing, or their interaction? **Not isolated.**
- **ACSEDebug:** Cross-game logging works; provenance, debug menu and native safety are unverified. Avoid the GUI.
- **Denuvo attribution:** The observed Sandboxie startup improvement does not prove the failure was caused by Denuvo or that `UnstrippedToken=y` bypasses anti-tamper.

## Recently answered (limited scope)

- **Can a custom ACSE component survive `startscreenglobe → tropicalworld`?** Yes, in the copied PC2 test installation, diagnostic 005 + park transition.
- **Are editor and undo APIs present in a loaded park?** Yes, diagnostic 006 returned `true` for the specified method-presence checks in `tropicalworld`. **Invoking those methods remains untested.**

Historical open questions and chronology are preserved in [LOG.md](LOG.md). Each newly answered question needs an actual runtime log or source anchor, with explicit limitations.
