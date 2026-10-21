# Open questions and next experiments

**Updated 2026-10-21 through diagnostic 034.** [Status](STATUS.md) · [Transport](../networking/TRANSPORT.md)

## Immediate blockers

1. **Transport:** What exact parameters construct CreateKeplerHttpRequest? Does it accept arbitrary/loopback URLs, and how is raw response text read? Function-valued access is verified; invocation is not.
2. **Two real clients:** Can separately authorized clients run concurrently, load matching baselines and exchange a harmless message/ack without game edits?
3. **Edit capture:** Where can our mod observe an ordinary player-confirmed create/move/delete without echoing remote edits or reacting to transient previews?
4. **Identity:** How will persistent mod-owned object keys map to each client's local PlacementPartID and transient EntityID, including reload and deletion tombstones?
5. **Recovery:** What authority, ownership, ordering, acknowledgement, durable command journal and resync rules handle disconnects, crashes and concurrent operations?

## Remaining coverage

- Version/executable/mod hashes, packaging and licensing for reproducible distribution.
- Multi-object groups, undo/redo, rotations/scales, different scenery and invalid placement/permission failures.
- Repeated persistence tests, stable canonical world inventories, save metadata for mod-owned mappings.
- Steam/profile/Cloud isolation and independently usable guest configuration.
- Native Lua module/bridge compatibility if built-in HTTP cannot provide the required transport.

## Answered within the tested scope

- Hierarchy creation and change-list access: diagnostics 007/008.
- Scripted AQ_Rock_04 creation, transform, preview, cancel, commit and world lookup: 009–018.
- Creation/movement/deletion persistence after manual reload: 019/022/025.
- Independence of two equal-resource parts: 026/027.
- Session-local duplicate suppression and ordering/ID-conflict rejection: 028/029.
- Canonical replay agreement across twelve inputs in separate A/B runs: 030.
- Lua io/os/LuaSocket unavailable in the tested environment: 031; no file roundtrip completed.
- Five named API fields resolve as functions while nine invented names resolve as nil: 033/034. Signatures and networking behavior remain unknown.

Historical failures remain in [LOG.md](LOG.md); they are not the current blocker. No playable co-op or live two-client synchronization is claimed.
