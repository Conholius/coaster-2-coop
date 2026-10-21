# Planet Coaster 2 — Real-Time Co-op Research

> **Status (2026-10-20): reverse engineering / feasibility research. No functioning multiplayer prototype and no verified persistent programmatic scenery placement.**

An independent research and prospective modding project investigating real-time collaborative park building in **Planet Coaster 2**. The initial scope is synchronizing user-confirmed scenery creation, movement and deletion across two copies of the game. Guest AI, economy, terrain, and ride simulation are **out of initial scope**.

## Findings at a glance

- PC2 exposes native and Lua-facing APIs for entities, placement identifiers, editing/undo, worlds, managers, and save/load orchestration.
- `Game.BaseGame` / `Game.GameScript` / `Game.GameEnvironment` reveal high-level game initialization and lifecycle.
- `ParkLoadSaveManager` delegates world save operations to the active world's `worldserialisation` interface. `LocalChangeManager` tracks local versions of online park **saves**, not granular editor mutations.
- `api.entity.InstantiatePrefab` demonstrably creates entities, but the evidence **does not show** that arbitrary spawned entities become durable scenery placements.
- Scenery-related specdefs describe `PlacementPartID`, resource names, transforms and serialization components, but they do not establish the editor's durable commit pathway.
- **New static finding:** The game's `SubModePathSnapPlacement` source shows scenery `CreateNewPart`, editing, preview validation, and a deferred `CommitPreview` after the player confirms placement. Details: [scenery research](docs/placement/README.md). Whether a mod can reproduce this and whether the edit survives save/reload remain untested.

## Start here

- [Research status and evidence conventions](docs/research/STATUS.md)
- [Lua, world and manager architecture](docs/architecture/OVERVIEW.md)
- [Placement and undo research](docs/placement/README.md)
- [Save/load and serialization](docs/serialization/README.md)
- [Modding tools and comparative references](docs/modding/README.md)
- [First prototype criteria and testing](docs/prototype/PLAN.md)
- [Isolated Windows/Sandboxie installation and compatibility notes](docs/ISOLATED-INSTALLATION.md)
- [File evidence index](docs/research/EVIDENCE.md)
- [Open questions](docs/research/OPEN_QUESTIONS.md)
- [Instructions for coding agents](AGENTS.md)

## Safety and licensing

This repository contains original research notes and, eventually, our own code, **not** dumped game files, OVL/OVS archives, full game decompilations, asset packages, save files, keys, or other proprietary redistributions. Raw research materials belong in the ignored local workspace. Respect licenses and terms; do not bypass anti-cheat or proprietary online service protections.

## Credits and acknowledgements

Thank you to the authors, maintainers and contributors of these community projects:

- **[ACSE — Advanced Cobra Script Extender](https://github.com/OpenNaja/ACSE)**, maintained in the OpenNaja repository, for expanding scripting capabilities in Frontier's Cobra-engine games and providing a valuable reference for scripting-extension research. See the [upstream README](https://github.com/OpenNaja/ACSE/blob/main/README.md) and [license](https://github.com/OpenNaja/ACSE/blob/main/LICENSE).
- **[Cobra Tools](https://github.com/OpenNaja/cobra-tools)** by OpenNaja and contributors, for making the extraction and inspection of Cobra-engine OVL/OVS archives and associated formats significantly easier. This tooling supported the file analysis that made the project's Lua decompilation research more manageable. See the [upstream README](https://github.com/OpenNaja/cobra-tools#readme) and [documentation](https://opennaja.github.io/cobra-tools/).
- **[ForgeUtils](https://github.com/Distantz/ForgeUtils)** by Distantz and contributors, for Planet Coaster 2 utility APIs, Lua and JavaScript hooking utilities, and custom scenery/ride builders. See the [upstream README](https://github.com/Distantz/ForgeUtils/blob/main/README.md), [documentation](https://forgeutils.distantz.com) and [license](https://github.com/Distantz/ForgeUtils/blob/main/LICENSE).

- **[Frontier Modding Club Discord](https://discord.gg/Su4jXKk)** — linked by [OpenNaja's ACSE project](https://github.com/OpenNaja/ACSE). Thanks to the community for access to the prefab descriptions and API reference materials used in this research. These materials are not redistributed here.

These are credited research references and candidate foundations. This repository currently contains research notes, not an integrated co-op implementation; no runtime integration, compatibility with the target PC2 build, persistent placement or multiplayer functionality is claimed. Dependency choices remain subject to testing and license review. Our repository's license does not replace upstream licenses.


## Collaboration

Inspired by [Co-op Anything](https://github.com/cranebeaker/coop-anything). Its [AGENTS.md](https://github.com/cranebeaker/coop-anything/blob/main/AGENTS.md) should be reviewed before contributing. The public game claim is tracked in [Co-op Anything issue #74](https://github.com/cranebeaker/coop-anything/issues/74). It describes the current feasibility-research stage; no functioning co-op prototype or persistent programmatic scenery-edit test has been demonstrated.

## Runtime compatibility warning — cross-game ACSEDebug (tested 2026-10-21)

**Do not assume the ACSEDebug package intended for Planet Zoo / earlier Frontier games is fully compatible with Planet Coaster 2.** In one isolated Windows PC2 test installation (game build **1.11.4.1011357**, ACSE **0.729**, ForgeUtils **2.7.0**), ACSEDebug **0.014** *did* produce a `Planet Coaster 2.log` and captured our original mod's `api.debug.Trace()` messages. This is **partial runtime compatibility**, **not** validation of its debug menu or every native feature. The exact cross-game package provenance should be independently checked before generalizing.

The same installation crashed on a new sandbox-park load with a custom ACSE `AddLuaComponents` world-diagnostic component enabled; disabling **that component registration**, while **keeping ACSEDebug enabled**, allowed the park to load and run for more than 60 seconds, then exit normally. Thus **the crash is not evidence that ACSEDebug alone prevents park loading**; the specific custom component lifecycle remains under investigation. The debug menu was **not tested in the successful park-load run** and should be treated as potentially unsafe. **Back up the isolated install, avoid the debug menu, and do not install cross-game debug binaries into a normal game installation.**

Detailed reproducible observations and remaining caveats: [research status](docs/research/STATUS.md), [research log](docs/research/LOG.md).
