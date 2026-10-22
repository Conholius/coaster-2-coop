# Planet Coaster 2 — Real-Time Co-op Research

> **Project status — 2026-10-22, reviewed through diagnostic 068:** Local native HTTP scenery commands, live-session Undo/Redo, ACK/reset/server-restart recovery and a controlled saved-checkpoint client restart are verified. A minimal registry passed with 2,701 real rocks and 1,464 moves; detailed timing and recovery limits are documented. **No playable co-op or simultaneous two-client synchronization yet.**

We are investigating how two players might build in the same **Planet Coaster 2** park in real time. The next network goal is a harmless message exchange between two simultaneously running clients, followed by replication of a validated scenery command. Sequential A/B exchange already works. Guests, rides, terrain, economy and full simulation synchronization remain outside the initial scope.

## Progress at a glance

| Milestone | Current evidence |
| --- | --- |
| Custom Lua mod loads | **Verified in PC2:** ACSE 0.729 loads the diagnostic mod; ForgeUtils 2.7.0's hook manager can be required. |
| ACSE world component survives park transition | **Verified:** minimal component with `Shutdown()` deactivates in `startscreenglobe` and is recreated in `tropicalworld` (diagnostic 005). |
| Editor and undo API visible in the park | **Verified:** diagnostic 006 found `worldAPIs.editors`, `CreateUndoOperationsHierarchy`, and four `api.undo` methods as functions in `tropicalworld`. **Presence checks only.** |
| Create an undo hierarchy | **Verified:** invoked successfully from our component (007/008). |
| Create, move, delete, save and reload a scenery part | **Verified for AQ_Rock_04:** diagnostics 009–025. |
| Ordered commands, duplicates and conflicts | **Verified locally:** 028/029; separate A/B replay states matched after twelve inputs in 030. |
| Native HTTP transport | **Verified:** owner-controlled loopback GET, HTTP status and text response via Kepler; active Frontier connection is not required once context is ready (037b–047). |
| Relay message exchange | **Verified sequentially:** A/B/A-again exchanged three messages each through a retained local server (038); simultaneous/internet test pending. |
| HTTP command application | **Verified in one client:** CREATE/MOVE/DELETE and Undo/Redo, with tested duplicate/order/ID-conflict guards (040–042, 055–061). |
| Grouped creation | **Verified for AQ_Rock_04:** 2,701 rocks / 13 commits, groups up to 500 (050–054). |
| Recovery | **Local controlled tests passed:** ACK refusal, TCP resets, journal-backed server restart and saved-checkpoint client restart (059–063); arbitrary crash safety remains open. |
| Native history after reload | **Unavailable in the tested restart (064).** Move counteractions work from manually transferred action data (065). |
| Registry and performance | **Passed:** synthetic minimal/full registry tests to 100,000 entries (066), real-object queries/moves with 2,701 rocks (067/068). |
| Send a scenery edit between two game clients | **Not implemented or tested.** |

The successful 005/006 logs came from a **copied test installation launched outside Sandboxie**. They establish PC2 behavior in that copied installation, **not** validation of the fully sandboxed launch path. A separate Sandboxie setup is documented below. See the [runtime test record](docs/testing/RUNTIME_TESTS.md).

## Where to start

- **[Documentation guide](docs/README.md)** — a map of the project by topic.
- **[Current research status](docs/research/STATUS.md)** — concise verified findings, limitations and next milestone.
- **[Runtime tests (005–068)](docs/testing/RUNTIME_TESTS.md)** — what was actually tested, failures and controls.
- **[Scenery placement and undo](docs/placement/README.md)** — the path toward creating a durable park object.
- **[Sandboxie test setup](docs/ISOLATED-INSTALLATION.md)** — separate host/guest boxes, token compatibility and isolation caveats.
- **[Prototype plan](docs/prototype/PLAN.md)** — acceptance criteria for one replicated scenery part.

For historical experiments and traceability: [research log](docs/research/LOG.md), [evidence index](docs/research/EVIDENCE.md), [open questions](docs/research/OPEN_QUESTIONS.md).

## What we know — and what we do not

Our component now uses the scenery hierarchy, create/edit/destroy operations, completed previews and single commits. Creation, movement and deletion were checked after local save/manual reload. Two identical rocks can be addressed independently. Commands reject tested ordering errors and conflicting ID reuse. See the [007–034 diagnostic ledger](docs/research/DIAGNOSTICS_007_034.md).

Diagnostic 030 matched canonical states without networking. Diagnostic 038 later exchanged actual messages through a server between sequential game runs; it still does not establish a simultaneous client pair. [Native HTTP transport](docs/networking/TRANSPORT.md) now works in the tested local setup. The next gaps are simultaneous/remote clients, confirmed user-edit capture and automatic durable recovery beyond the completed-checkpoint tests.

Read the [050–068 evidence ledger](docs/research/DIAGNOSTICS_050_068.md), [recovery/history findings](docs/networking/RECOVERY_AND_HISTORY.md), [035–049 evidence ledger](docs/research/DIAGNOSTICS_035_049.md), [transport/editor measurements](docs/networking/BENCHMARKS.md) and [proposed HTTP synchronization design](docs/networking/HTTP_SYNC_DESIGN.md). The architecture is a proposal, not implemented end-to-end synchronization.

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
| Command transport | [Native HTTP contract and service dependence](docs/networking/TRANSPORT.md) |
| Proposed replication | [HTTP synchronization, identity and recovery](docs/networking/HTTP_SYNC_DESIGN.md) |
| Performance | [HTTP, editor, registry and query measurements](docs/networking/BENCHMARKS.md) |
| Recovery and history | [ACKs, checkpoints and counteractions](docs/networking/RECOVERY_AND_HISTORY.md) |
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
