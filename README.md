# Planet Coaster 2 — Real-Time Co-op Research

> **Project status — 2026-10-21, diagnostics 007–034:** Scripted scenery creation, movement, deletion and local save/reload persistence are verified. Ordered replay and duplicate/conflict checks work in the tested local scenarios. **No playable co-op mod or live two-client synchronization yet.**

We are investigating how two players might build in the same **Planet Coaster 2** park in real time. The next goal is a harmless message exchange between two genuine clients, followed by replication of a validated scenery command. Guests, rides, terrain, economy and full simulation synchronization remain outside the initial scope.

## Progress at a glance

| Milestone | Current evidence |
| --- | --- |
| Custom Lua mod loads | **Verified in PC2:** ACSE 0.729 loads the diagnostic mod; ForgeUtils 2.7.0's hook manager can be required. |
| ACSE world component survives park transition | **Verified:** minimal component with `Shutdown()` deactivates in `startscreenglobe` and is recreated in `tropicalworld` (diagnostic 005). |
| Editor and undo API visible in the park | **Verified:** diagnostic 006 found `worldAPIs.editors`, `CreateUndoOperationsHierarchy`, and four `api.undo` methods as functions in `tropicalworld`. **Presence checks only.** |
| Create an undo hierarchy | **Verified:** invoked successfully from our component (007/008). |
| Create, move, delete, save and reload a scenery part | **Verified for AQ_Rock_04:** diagnostics 009–025. |
| Ordered commands, duplicates and conflicts | **Verified locally:** 028/029; separate A/B replay states matched after twelve inputs in 030. |
| Lua transport access | **Investigation 031–034:** candidate HTTP/Kepler functions resolve; signatures, free URLs and raw response access remain unknown. |
| Send a scenery edit between two game clients | **Not implemented or tested.** |

The successful 005/006 logs came from a **copied test installation launched outside Sandboxie**. They establish PC2 behavior in that copied installation, **not** validation of the fully sandboxed launch path. A separate Sandboxie setup is documented below. See the [runtime test record](docs/testing/RUNTIME_TESTS.md).

## Where to start

- **[Documentation guide](docs/README.md)** — a map of the project by topic.
- **[Current research status](docs/research/STATUS.md)** — concise verified findings, limitations and next milestone.
- **[Runtime tests (005–034)](docs/testing/RUNTIME_TESTS.md)** — what was actually tested, failures and controls.
- **[Scenery placement and undo](docs/placement/README.md)** — the path toward creating a durable park object.
- **[Sandboxie test setup](docs/ISOLATED-INSTALLATION.md)** — separate host/guest boxes, token compatibility and isolation caveats.
- **[Prototype plan](docs/prototype/PLAN.md)** — acceptance criteria for one replicated scenery part.

For historical experiments and traceability: [research log](docs/research/LOG.md), [evidence index](docs/research/EVIDENCE.md), [open questions](docs/research/OPEN_QUESTIONS.md).

## What we know — and what we do not

Our component now uses the scenery hierarchy, create/edit/destroy operations, completed previews and single commits. Creation, movement and deletion were checked after local save/manual reload. Two identical rocks can be addressed independently. Commands reject tested ordering errors and conflicting ID reuse. See the [007–034 diagnostic ledger](docs/research/DIAGNOSTICS_007_034.md).

Separate A/B-labelled runs produced matching canonical states, but did not exchange messages or run as a demonstrated simultaneous client pair. The present blocker is [command transport](docs/networking/TRANSPORT.md), followed by confirmed user-edit capture, identity mapping and recovery.

An earlier custom world component caused a crash during the menu-to-park transition. A later minimal component that included `Shutdown()` passed that transition; because other probes were removed at the same time, **we cannot attribute the fix solely to `Shutdown()`**. Full history: [runtime tests](docs/testing/RUNTIME_TESTS.md).

## Compatibility and safety

The runtime logs identify **PC2 1.11.4.1011357**, **ACSE 0.729**, **ForgeUtils 2.7.0** and **ACSEDebug 0.014**. The cross-game ACSEDebug package captured log messages, but its **debug GUI and native helpers are not verified safe in PC2**; avoid the GUI. The exact package provenance should not be inferred from the logs.

Use only backed-up, disposable installs and test parks. The [Sandboxie notes](docs/ISOLATED-INSTALLATION.md) describe a past startup compatibility setting, `UnstrippedToken=y`, which **reduces token isolation**. It does not prove a Denuvo defect or establish full cloud/save isolation. Do not use the old diagnostic launcher without checking its mod-restoration side effects.

This repository contains our own notes, not redistributed game binaries, assets, dumps, OVL/OVS files, decompiled scripts, private logs or credentials. Respect third-party licenses and service protections; do not bypass anti-cheat.

## Project reference pages

| Area | Page |
| --- | --- |
| Lua / world architecture | [Architecture](docs/architecture/OVERVIEW.md) |
| Editing / undo / creation | [Placement](docs/placement/README.md) and [ACSE + ForgeUtils research](docs/placement/ACSE_FORGEUTILS.md) |
| Command transport | [HTTP/Kepler and Lua capability investigation](docs/networking/TRANSPORT.md) |
| Saving / persistence | [Serialization](docs/serialization/README.md) |
| Tools and decompilation | [Modding references](docs/modding/README.md) |
| Contribution / evidence rules | [AGENTS.md](AGENTS.md) |
| Co-op Anything inspiration | [Co-op Anything](https://github.com/cranebeaker/coop-anything) and [PC2 research issue #74](https://github.com/cranebeaker/coop-anything/issues/74) |

## Credits and acknowledgements

- **[ACSE — Advanced Cobra Script Extender](https://github.com/OpenNaja/ACSE)** (OpenNaja and contributors) — Lua scripting-extension work; [license](https://github.com/OpenNaja/ACSE/blob/main/LICENSE).
- **[Cobra Tools](https://github.com/OpenNaja/cobra-tools)** (OpenNaja and contributors) — Cobra archive and file research; [documentation](https://opennaja.github.io/cobra-tools/).
- **[ForgeUtils](https://github.com/Distantz/ForgeUtils)** (Distantz and contributors) — PC2 Lua/JavaScript tooling and catalogue builders; [documentation](https://forgeutils.distantz.com), [license](https://github.com/Distantz/ForgeUtils/blob/main/LICENSE).
- **[Frontier Modding Club](https://discord.gg/Su4jXKk)** — linked by the ACSE project; community reference material is credited, **not redistributed**.

These are research dependencies and comparative examples, not endorsements or evidence of finished co-op.
