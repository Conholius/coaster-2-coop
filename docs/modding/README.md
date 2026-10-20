# Modding and research references

## Project-specific material studied

- [Co-op Anything](https://github.com/cranebeaker/coop-anything) — guidance, QA and contributions.
- [Cobra Tools](https://github.com/OpenNaja/cobra-tools) — Cobra-engine archive/file analysis; support for the exact PC2 target formats should be independently validated.
- Enhanced Move Tool (PC2) — editor gizmo and transform hooks; reviewed code does **not** prove durable final placement commits.
- ProTrack (PC2) — track-specific editor preview hooks; not direct scenery-edit proof.
- Comparative mods from other Frontier games (ACSE, toolboxes/free-build mods) — possible patterns only; no cross-game API transfer without PC2 evidence.

## Technical caution

PC2 Lua bytecode is Lua 5.3-like but carries a custom format byte and a shifted header versus stock Lua 5.3. A conversion (changing header format byte `0x02` to `0x00` and inserting a four-byte instruction-size field value at header position 14) allowed unluac to parse many original files. Not universal: `debuglistener.lua.bin` triggered an unmapped constant type error, and `FrontierAccountManager` failed source reconstruction despite successful full disassembly. Validate every important decompilation.

Keep actual game files and extracted full scripts only in ignored research storage. Do not upload their content to this repository.
