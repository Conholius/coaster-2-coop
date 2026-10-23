# Documentation guide

[← Project overview](../README.md) · [Current status](research/STATUS.md) · [Runtime evidence](testing/RUNTIME_TESTS.md)

The project is research-first. Read **current results** before **historical investigations**: older pages record failed hypotheses and planned steps that later tests superseded.

## Start here

| Your question | Read |
| --- | --- |
| Does the co-op mod already work? | [Project README](../README.md) and [current status](research/STATUS.md) |
| Which PC2 calls actually worked? | [Runtime tests and diagnostics](testing/RUNTIME_TESTS.md) |
| What is the tested local host/guest setup? | [Sandboxie installation notes](ISOLATED-INSTALLATION.md) |
| Why are some requests slow? | [080–082 latency investigation](networking/HTTP_LATENCY_080_082.md) |
| What is the next experiment? | [Current open questions](research/OPEN_QUESTIONS.md) |
| What is still unknown? | [Open questions](research/OPEN_QUESTIONS.md) |

## Technical areas

| Topic | Page | Type of evidence |
| --- | --- | --- |
| Editor hierarchy, scenery creation and undo | [Scenery placement](placement/README.md) | Runtime create/edit/delete, preview/cancel/commit |
| Existing ACSE / ForgeUtils approaches | [Placement research](placement/ACSE_FORGEUTILS.md) | Commit-pinned public source research |
| Lua, world and manager lifecycle | [Architecture](architecture/OVERVIEW.md) | Inspected source; lifecycle partly runtime-tested |
| Save/load and object persistence | [Serialization](serialization/README.md) | Local persistence and controlled saved-checkpoint client reload |
| Recovery and Undo history | [Recovery findings](networking/RECOVERY_AND_HISTORY.md) | ACK/reset/server recovery, checkpoint reload, native-history loss and Move counteractions |
| Dependencies, bytecode and tooling | [Modding](modding/README.md) | Source research + limited runtime loading |
| Command transport | [Native HTTP](networking/TRANSPORT.md) | Native GET/text responses, readiness and disconnect controls |
| Synchronization design | [Host/relay protocol](networking/HTTP_SYNC_DESIGN.md) | Proposed complete architecture; scripted two-client and single manual Create paths verified |
| Performance | [Measurements](networking/BENCHMARKS.md) | HTTP, grouped edits, registry/query profiling and two-client native/Python controls through 082 |
| Two-client acceptance criteria | [Prototype plan](prototype/PLAN.md) | Completed diagnostic gates and remaining product work |

## Evidence and chronology

- [**STATUS.md**](research/STATUS.md): short, current facts and limitations.
- [**EVIDENCE.md**](research/EVIDENCE.md): where each claim comes from and what it does **not** prove.
- [**LOG.md**](research/LOG.md): chronological experiments, including failures and superseded hypotheses.
- [**OPEN_QUESTIONS.md**](research/OPEN_QUESTIONS.md): active blockers, in priority order.
- [**Runtime diagnostics**](testing/RUNTIME_TESTS.md): checkpoints through 082; ledgers [007–034](research/DIAGNOSTICS_007_034.md), [035–049](research/DIAGNOSTICS_035_049.md), [050–068](research/DIAGNOSTICS_050_068.md) and [069–082](research/DIAGNOSTICS_069_082.md).

## Ground rules

Do not confuse **a function being present** with **a successful invocation**, **a runtime entity** with **a saved scenery placement**, or **a copied game installation** with **Sandboxie isolation**. Keep logs, personal paths, game binaries and saves private. See [AGENTS.md](../AGENTS.md).
