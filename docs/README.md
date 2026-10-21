# Documentation guide

[← Project overview](../README.md) · [Current status](research/STATUS.md) · [Runtime evidence](testing/RUNTIME_TESTS.md)

The project is research-first. Read **current results** before **historical investigations**: older pages record failed hypotheses and planned steps that later tests superseded.

## Start here

| Your question | Read |
| --- | --- |
| Does the co-op mod already work? | [Project README](../README.md) and [current status](research/STATUS.md) |
| Which PC2 calls actually worked? | [Runtime tests and diagnostics](testing/RUNTIME_TESTS.md) |
| How is the isolated host/guest setup intended to work? | [Sandboxie installation notes](ISOLATED-INSTALLATION.md) |
| What is the next experiment? | [Runtime tests: diagnostic 007](testing/RUNTIME_TESTS.md#next-experiment-diagnostic-007) |
| What is still unknown? | [Open questions](research/OPEN_QUESTIONS.md) |

## Technical areas

| Topic | Page | Type of evidence |
| --- | --- | --- |
| Editor hierarchy, scenery creation and undo | [Scenery placement](placement/README.md) | Static scenery call-path + runtime API presence |
| Existing ACSE / ForgeUtils approaches | [Placement research](placement/ACSE_FORGEUTILS.md) | Commit-pinned public source research |
| Lua, world and manager lifecycle | [Architecture](architecture/OVERVIEW.md) | Inspected source; lifecycle partly runtime-tested |
| Save/load and object persistence | [Serialization](serialization/README.md) | Inspected source; no mod-created persistence proof |
| Dependencies, bytecode and tooling | [Modding](modding/README.md) | Source research + limited runtime loading |
| Two-client acceptance criteria | [Prototype plan](prototype/PLAN.md) | Planned; not yet demonstrated |

## Evidence and chronology

- [**STATUS.md**](research/STATUS.md): short, current facts and limitations.
- [**EVIDENCE.md**](research/EVIDENCE.md): where each claim comes from and what it does **not** prove.
- [**LOG.md**](research/LOG.md): chronological experiments, including failures and superseded hypotheses.
- [**OPEN_QUESTIONS.md**](research/OPEN_QUESTIONS.md): active blockers, in priority order.
- [**Runtime diagnostics**](testing/RUNTIME_TESTS.md): reproducible checkpoints (005, 006) and planned 007.

## Ground rules

Do not confuse **a function being present** with **a successful invocation**, **a runtime entity** with **a saved scenery placement**, or **a copied game installation** with **Sandboxie isolation**. Keep logs, personal paths, game binaries and saves private. See [AGENTS.md](../AGENTS.md).
