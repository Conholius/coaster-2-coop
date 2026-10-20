# Planet Coaster 2 — Real-Time Co-op Research

> **Status (2026-10-20): reverse engineering / feasibility research. No functioning multiplayer prototype and no verified persistent programmatic scenery placement.**

An independent research and prospective modding project investigating real-time collaborative park building in **Planet Coaster 2**. The initial scope is synchronizing user-confirmed scenery creation, movement and deletion across two copies of the game. Guest AI, economy, terrain, and ride simulation are **out of initial scope**.

## Findings at a glance

- PC2 exposes native and Lua-facing APIs for entities, placement identifiers, editing/undo, worlds, managers, and save/load orchestration.
- `Game.BaseGame` / `Game.GameScript` / `Game.GameEnvironment` reveal high-level game initialization and lifecycle.
- `ParkLoadSaveManager` delegates world save operations to the active world's `worldserialisation` interface. `LocalChangeManager` tracks local versions of online park **saves**, not granular editor mutations.
- `api.entity.InstantiatePrefab` demonstrably creates entities, but the evidence **does not show** that arbitrary spawned entities become durable scenery placements.
- Scenery-related specdefs describe `PlacementPartID`, resource names, transforms and serialization components, but they do not establish the editor's durable commit pathway.
- The crucial unknown: how a normal confirmed scenery edit moves from editor preview through change list / undo into persistent park placement data.

## Start here

- [Research status and evidence conventions](docs/research/STATUS.md)
- [Lua, world and manager architecture](docs/architecture/OVERVIEW.md)
- [Placement and undo research](docs/placement/README.md)
- [Save/load and serialization](docs/serialization/README.md)
- [Modding tools and comparative references](docs/modding/README.md)
- [First prototype criteria and testing](docs/prototype/PLAN.md)
- [File evidence index](docs/research/EVIDENCE.md)
- [Open questions](docs/research/OPEN_QUESTIONS.md)
- [Instructions for coding agents](AGENTS.md)

## Safety and licensing

This repository contains original research notes and, eventually, our own code, **not** dumped game files, OVL/OVS archives, full game decompilations, asset packages, save files, keys, or other proprietary redistributions. Raw research materials belong in the ignored local workspace. Respect licenses and terms; do not bypass anti-cheat or proprietary online service protections.

## Credits and acknowledgements

Thank you to the authors, maintainers and contributors of these community projects:

- **[ACSE — Advanced Cobra Script Extender](https://github.com/OpenNaja/ACSE)**, maintained in the OpenNaja repository, for expanding scripting capabilities in Frontier's Cobra-engine games and providing a valuable reference for scripting-extension research. See the [upstream README](https://github.com/OpenNaja/ACSE/blob/main/README.md) and [license](https://github.com/OpenNaja/ACSE/blob/main/LICENSE).
- **[ForgeUtils](https://github.com/Distantz/ForgeUtils)** by Distantz and contributors, for Planet Coaster 2 utility APIs, Lua and JavaScript hooking utilities, and custom scenery/ride builders. See the [upstream README](https://github.com/Distantz/ForgeUtils/blob/main/README.md), [documentation](https://forgeutils.distantz.com) and [license](https://github.com/Distantz/ForgeUtils/blob/main/LICENSE).

These are credited research references and candidate foundations. This repository currently contains research notes, not an integrated co-op implementation; no runtime integration, compatibility with the target PC2 build, persistent placement or multiplayer functionality is claimed. Dependency choices remain subject to testing and license review. Our repository's license does not replace upstream licenses.

References checked against the upstream repositories on 2026-10-20.

## Collaboration

Inspired by [Co-op Anything](https://github.com/cranebeaker/coop-anything). Its [AGENTS.md](https://github.com/cranebeaker/coop-anything/blob/main/AGENTS.md) should be reviewed before contributing. The proposed public game claim is captured in [CLAIM.md](CLAIM.md); the submission attempt on 2026-10-20 returned HTTP 403 (`Resource not accessible by integration`), so no issue creation is confirmed.
