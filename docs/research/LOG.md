# Research log

## 2026-10-20 — Consolidation

- Organized prior static research on PC2 Lua environments, editor registries, undo API names, entity spawning, save/load orchestration and component schema material.
- Distinction reaffirmed: an entity spawned using `InstantiatePrefab` is not known to be a persisted park placement.
- `LocalChangeManager` disambiguated: manages local versions of online park saves, not per-object collaborative editing events.
- Key next step identified: find confirmed scenery edit commit call site and test save/reload persistence on an isolated park.
- No successful real-time synchronization, multiplayer code or live in-game validation yet.

Future entries: record date, test build + SHA-256, original symbol/source, test procedure, observed behavior, failure mode and remaining uncertainty.

## 2026-10-20 — ACSE / ForgeUtils placement investigation

- Inspected pinned upstream sources: ForgeUtils catalogue builder and hook lifecycle, ACSE entity serialization, and PC-CC's PC2 undo preview/commit sequence.
- Confirmed in source that SceneryPartBuilder writes catalogue database rows rather than creating park instances.
- Found a concrete PC2 preview/wait/check/commit/wait example; its operations are track-specific, so scenery creation remains unproven.
- Documented the exact missing bridge and reload/duplicate-control experiment in [the placement investigation](../placement/ACSE_FORGEUTILS.md).
- No runtime tests performed; local game dumps and an isolated test environment are not available in the current workspace.
