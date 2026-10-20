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

## Collaboration

Inspired by [Co-op Anything](https://github.com/cranebeaker/coop-anything). Its [AGENTS.md](https://github.com/cranebeaker/coop-anything/blob/main/AGENTS.md) should be reviewed before contributing. The proposed public game claim is captured in [CLAIM.md](CLAIM.md); claim submission is not yet confirmed.
