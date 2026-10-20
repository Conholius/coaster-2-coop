# Game, Lua, world and manager architecture

## Lua modules

The game uses `require` and `module(...)` patterns; its API is partly native and partly Lua-wrapped. `API.Debug` exposes `GetListOfLoadedModuleNames()` (reads `package.loaded`) and `ReloadModule()` (uses `loadresource`, `tryrequire`, `pcall`). The game registers development shell commands for listing/reloading Lua modules when `api.debug.IsDebugAllowed()` permits them. Availability in retail remains **unknown**.

`Common.ModuleUtils` provides debug-gated metatable checks (`__index` / `__newindex`); it **does not** enumerate resources or map modules to OVL files. Logical module paths need not match extracted resource filenames.

## Game startup and environment

Observed chain: `Game.GameScript` configures `Game.GameEnvironment`; `Game.BaseGame` initializes game APIs/environment and handles world transitions; `Environment.Environment` creates dependency-managed managers and phase callbacks. `Game.GameEnvironment` references managers including `ParkLoadSaveManager`, `LocalChangeManager`, `WorkshopManager`, and `NextWorldDataManager`.

`api.game.GetEnvironment()` accesses the game-wide environment. World-local manager access can involve `api.world.GetScript(api.world.GetActive()).tEnvironment`. Keep these scopes separate.

## Modes

`GameEditors` and UI mode registries refer to logical modules such as `Editors.Scenery.SceneryEditMode`, `Editors.Shared.MultiFeatureMoveMode`, `Editors.Select.SelectMode` and `Editors.Blueprint.BlueprintPlacementMode`. `GameHelpers.ModeHelper` demonstrates a mode pattern using `require`, instantiation, initialization, transition, update and requested interfaces. **Do not assume every PC2 editor mode uses this helper** without tracing the caller.

## Entities and transforms

`CommonObjectCreator` creates models, lights and environment capture groups via `api.entity.InstantiatePrefab`. Parameters include prefab name, instance name, completion token, `TransformQ`, parent, local-space flag and public properties. `TransformPrefab` appears as a parent for other instantiated entities. These are entities, not necessarily persisted scenery placements.

## Coroutines

`Common.CoroutineUtils` provides scheduling/completion helpers including `Parallel`; they coordinate Lua coroutines, not necessarily execution across CPU cores. Important decompiler output should be checked against bytecode.
