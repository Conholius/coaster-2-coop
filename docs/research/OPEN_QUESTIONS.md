# Open questions (priority order)

1. **Durable editor commit:** The scenery editor pathway is now visible in `SubModePathSnapPlacement` and brush placement. Can we verify its complete arguments and completion ordering, and expose the same confirmed commit operations through a permitted mod?
2. **Reproducible placement:** Can a permitted mod invoke the exact operation in a running retail build and does the result survive save/reload?
3. **Change detection:** Where can we observe only user-confirmed edits rather than previews, transient transforms or drag updates?
4. **Identifier translation:** How are EntityID, PlacementID, PlacementPartID and saved IDs assigned, mapped and replaced when actions are replayed across different clients?
5. **World serialization implementation:** Which native or scripted module provides `api.world.GetWorldAPIs().worldserialisation`? Is it reachable for loaded-park inspection?
6. **Park environment:** Which modules and managers are registered in the active park-specific world environment?
7. **Logical module resolution:** How does `loadresource("Editors.Scenery.SceneryEditMode")` map to actual packaged OVL/OVS resources?
8. **Session baseline:** Can identical initial park state be loaded independently and hashes/placement inventories compared?
9. **Network authority:** Which operation validation, order, conflict handling, retransmission and resynchronization model works without corrupting a park?
10. **Safety/versioning:** What legal/mod loader constraints, exact build versions, provenance and distribution restrictions apply?

An answer requires a source reference and, for runtime claims, a recorded test. Do not mark an item solved by inference alone.

## Narrowing from public source research (2026-10-20)

[PC2 mod source now demonstrates the general preview/commit lifecycle](../placement/ACSE_FORGEUTILS.md), including operation-token waiting. Additional PC2 scenery editor Lua now supplies concrete scenery creation/edit/clone and deferred commit call sites. Question 1 is narrowed to runtime invocation and reliable event interception. Question 2 still requires save/reload tests. ForgeUtils catalogue registration is not that missing operation. ACSE's serializer must be checked against the actual PC2 package before considering it as an alternate persistence path.

## Runtime diagnostics / component lifecycle (2026-10-21)

- **ACSEDebug compatibility:** A cross-game ACSEDebug 0.014 package successfully logged from PC2 1.11.4.1011357 with ACSE 0.729, but are its debug menu and native helpers safe? **Unknown; avoid debug UI.** Exact installed cross-game package provenance and hashes are not yet independently documented. Do not recommend unconditional installation.
- **Custom component park-transition crash:** Why does the PC2CoopDiagnosticComponent registered with `AddLuaComponents` crash during `startscreenglobe -> tropicalworld`? A successful control disabled that registration while keeping ACSEDebug present. Investigate expected `Shutdown`, `OnWorldDeactivation`, recreation and manager registration contracts, without assuming one missing callback caused the crash.
- **Editor presence vs invocation:** Editor / undo methods existed during `startscreenglobe` activation; are these APIs also safely callable after the actual `tropicalworld` has activated? Currently **untested**.
