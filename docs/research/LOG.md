# Research log

## 2026-10-20 — Consolidation

- Organized prior static research on PC2 Lua environments, editor registries, undo API names, entity spawning, save/load orchestration and component schema material.
- Distinction reaffirmed: an entity spawned using `InstantiatePrefab` is not known to be a persisted park placement.
- `LocalChangeManager` disambiguated: manages local versions of online park saves, not per-object collaborative editing events.
- Key next step identified: find confirmed scenery edit commit call site and test save/reload persistence on an isolated park.
- No successful real-time synchronization, multiplayer code or live in-game validation yet.

Future entries: record date, test build + SHA-256, original symbol/source, test procedure, observed behavior, failure mode and remaining uncertainty.
