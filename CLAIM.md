# Co-op Anything: proposed claim issue

**Status:** Prepared text only. The submission attempt via the connected GitHub integration returned HTTP 403 (`Resource not accessible by integration`) on 2026-10-20; **no issue was confirmed created**. The connected account can read the upstream repository but the integration could not create this issue. No matching Planet Coaster issue or PR was found before submission.

**Issue title:** Claim a game: Planet Coaster 2 (real-time co-op park building)

### Game

Planet Coaster 2 (PC, Frontier Developments)

### Plan

Starting from scratch: reverse engineering the PC2 Lua API, editor/placement operations and world serialization to prototype real-time collaborative park building. The initial focus is user-confirmed scenery placement, movement and deletion across two game clients.

Research to date: game/environment manager lifecycle, PC2 Lua bytecode, editor/undo API symbol inventory, save/load orchestration through `worldserialisation`, the distinction between online local save versions and per-object changes, and scenery/terrain serialization component definitions.

**No functioning co-op prototype or persistent programmatic scenery-edit test has been demonstrated.** The next milestone is to trace and replay a regular scenery commit, save/reload to validate persistence, and then attempt two-client synchronization. Research notes and owned mod code will live in the public project repository, without redistributing proprietary game dumps.

### Your repo or branch (optional)

https://github.com/Conholius/coaster-2-coop

**Official template:** https://github.com/cranebeaker/coop-anything/issues/new?template=claim-a-game.yml
