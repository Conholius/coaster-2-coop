# Modding and research references

## Project-specific material studied

- [Co-op Anything](https://github.com/cranebeaker/coop-anything) — guidance, QA and contributions.
- [Cobra Tools](https://github.com/OpenNaja/cobra-tools) — Cobra-engine archive/file analysis; support for the exact PC2 target formats should be independently validated.
- Enhanced Move Tool (PC2) — editor gizmo and transform hooks; reviewed code does **not** prove durable final placement commits.
- ProTrack (PC2) — track-specific editor preview hooks; not direct scenery-edit proof.
- [ACSE — Advanced Cobra Script Extender](https://github.com/OpenNaja/ACSE) — credit to OpenNaja and upstream contributors for Cobra scripting-extension work. Comparative reference; no cross-game API transfer or target-build compatibility without PC2 evidence.
- [ForgeUtils](https://github.com/Distantz/ForgeUtils) — credit to Distantz and contributors for PC2 utility APIs, Lua/JavaScript hooks and scenery/ride builders; [upstream documentation](https://forgeutils.distantz.com). Candidate foundation, not a verified integration or proof of durable editor placement.
- Comparative toolboxes/free-build mods from other Frontier games — possible patterns only; no cross-game API transfer without PC2 evidence.

Upstream README references for ACSE and ForgeUtils were checked on 2026-10-20. See the [project credits](../../README.md#credits-and-acknowledgements) for license links and the limits of these acknowledgements.

## Technical caution

PC2 Lua bytecode is Lua 5.3-like but carries a custom format byte and a shifted header versus stock Lua 5.3. A conversion (changing header format byte `0x02` to `0x00` and inserting a four-byte instruction-size field value at header position 14) allowed unluac to parse many original files. Not universal: `debuglistener.lua.bin` triggered an unmapped constant type error, and `FrontierAccountManager` failed source reconstruction despite successful full disassembly. Validate every important decompilation.

Keep actual game files and extracted full scripts only in ignored research storage. Do not upload their content to this repository.
