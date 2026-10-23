# Planet Coaster 2 in Sandboxie

> English translation of the project's German `ISOLIERTE-INSTALLATION.md` setup notes (2026-10-21). This documents the **specific local test arrangement**, not a universally verified installation procedure or an anti-tamper workaround.

## Launch

Double-click **Planet Coaster 2 - Isoliert.cmd**.

First, close the regular Steam client using **Steam > Exit**. The launcher opens Steam in the `PC2CoopHost` sandbox and waits only until Steam is signed in and online. If Steam requests authentication, sign in manually in the sandboxed Steam client, then run the launcher again.

## Layout

The following paths are relative to the local workspace; their precise absolute locations depend on that installation.

- `installation/work/coop-anything/local/test-install`: a separate copy of the game.
- `installation/work/coop-anything/local/sandboxes/PC2CoopHost`: redirected files, settings and Steam state for the test environment.
- `installation/work/coop-anything/local/sandboxes/PC2CoopGuest`: a separate spare sandbox with networking blocked; **no second game client has been validated**.
- `installation/work/coop-anything/local/backups`: backups of mod files.
- `installation/work/coop-anything/local/installation-manifest.json`: setup state and the checked game version.

The installation is a copy of game build **1.11.4.1011357**, with **ACSE 0.729** and the mods previously present. The normal launcher restores the original mod files from before the diagnostic intervention. It **does not install a co-op mod**. Diagnostics and networked co-op are still development work.

## Configuration and limitations

`UnstrippedToken=y` was necessary for a successful launch in **this tested Sandboxie configuration**. It reduces Windows token isolation. File redirection and Sandboxie access rules remain active; both sentinel-file isolation checks are rerun before launch. **Those checks do not prove isolation of every game or cloud feature.**

Access to the original game installation on drive `E:` is blocked for both sandboxes. The three known game-profile directories are isolated. Automatic recovery and automatic deletion are disabled. Earlier exceptions allowing communication with the regular Steam client have been removed. Steam, its WebHelper and the game have network access in the host sandbox. DLSS Enabler and Millennium are hidden only from the relevant sandboxed processes; original files are not renamed.

Keep **Steam Cloud disabled for Planet Coaster 2 in the sandboxed Steam client**. Previously existing cloud saves may already be present as local copies. Use a fresh test park for mod testing. Do not restore saves from the sandbox into the regular profile unless that is explicitly intended.

The `SBIE2205` message also appeared during successful launches; **it is not, by itself, proof of a launch failure**. Reaching the main menu was confirmed with this compatibility configuration. At the time of these original setup notes, scenery placement, saving and multiplayer behavior had **not** been validated by this sandbox-launch test.

The old directories were not deleted. **If this project folder is moved, its absolute Sandboxie paths must be configured again.**

## Security and evidence boundary

This configuration prioritizes compatibility while retaining selected filesystem restrictions. `UnstrippedToken=y` weakens an aspect of process isolation; do not present the sandbox as a security boundary with complete coverage. The observed startup improvement does **not** establish that Denuvo Anti-Tamper caused the original failure, or that the configuration bypasses Denuvo. Preserve backups and avoid running experiments against the regular profile.

This document describes the user's local setup and reported tests. It intentionally excludes personally identifying absolute paths, log files, binaries, credentials, and proprietary game content.

## Relationship to later editor diagnostics (2026-10-21)

**Important:** The later successful diagnostic 005/006 sessions used the copied `test-install` but **were not launched inside Sandboxie**. The tester noticed this after those sessions. Therefore the observed custom-component lifecycle and editor API method-presence results are valid for the copied installation but **have not been repeated under the configured Sandboxie host**.

The archived launch scripts may restore a previous baseline mod or change `Mod_Degridded/Main.ovl`; **inspect them before reusing them for new diagnostics** so that the intended `PC2CoopDiagnostics/Main.ovl` is not replaced or bypassed. The file `Planet Coaster 2 - Isoliert.cmd` belongs to the original setup and is not itself proof that a newly-built diagnostic mod was loaded.

See [runtime test matrix](testing/RUNTIME_TESTS.md) for the strict separation of confirmed game observations and planned test 007.

## Later validation — 2026-10-23

This page's earlier unvalidated-Guest statements describe the original setup date. Later owner-run tests launched Host and Guest together in the configured separate arrangement and passed two-client loopback messaging, scripted scenery/recovery and one manual host Create (071–081). A successfully started Guest Steam/game supersedes the original startup blocker for this arrangement; the 0x3000 startup issue was resolved after correcting the owner's network permissions, not attributed to a proven Steam/anti-tamper defect.

082 additionally ran Python inside PC2CoopGuest using Sandboxie's explicit box launcher. The Python process checked SbieDll injection, then completed 120 direct socket probes while both game processes stayed alive. Python must have its own allowed network access; game-process access does not imply Python access. Injection confirms Sandboxie presence, not by itself the exact box name or every permission boundary.

Start the diagnostic relay first, the Python control in the named Guest box, then both games/tropical parks. Leave all participants alive until every phase/final completion. Restarting one participant against an already advanced 082 session is unsupported; a fresh run requires all participants and server to restart.

This is same-machine testing, not a requirement for the final product's lobby or guest-local saving. It does not demonstrate complete Steam/Cloud/account/save isolation, remote access, licensing portability or service-shutdown independence. [Latest evidence](research/DIAGNOSTICS_069_082.md) · [HTTP controls](networking/HTTP_LATENCY_080_082.md).
