# Architecture: RogueCellar

This document is aimed at **maintainers** and focuses on the structure of the codebase and the reasoning behind it
(especially the use of **effects and handlers**).

## Goals

- Keep the game logic **testable** by isolating I/O behind effects.
- Make randomness **deterministic** when given a seed.
- Keep modules small and focused so it’s easy to extend (new tiles, new monsters, new rules).


## Module map

```
src/
  main.effekt            Entrypoint; calls runtime::runGame()
  test.effekt            Test harness + unit tests

  rogue/
    model.effekt         Types + effect interfaces
    util.effekt          Tiny helpers (list ops, math helpers)
    dungeon.effekt       Map generation + tile accessors
    logic.effekt         Rules: movement, combat, AI, turns
    render.effekt        Rendering + input parsing
    runtime.effekt       Handlers (GameState/Rng/Log/UI/Controller) + bootstrap loop
```

### “Core vs boundary” split

- `dungeon.effekt`, `logic.effekt`, and `render.effekt` are the **core**.
  - They are mostly pure, and when they need state/randomness/logging they do it through *effects*.
- `runtime.effekt` is the **boundary**.
  - It provides concrete handlers that interpret those effects using `Ref`s and console I/O.

## Key data types

Defined in `src/rogue/model.effekt`:

- `Dungeon`: width/height and a grid of `Tile` (walls/floor/exit).
- `World`: the whole game state (dungeon, player, monsters, status, turn, difficulty).
- `MonsterKind`: currently `Goblin` and `Rat`.
- `Action`: what an actor does each turn (`Move`, `Wait`, `QuitCmd`).

## Effects and handlers

The project models side effects explicitly, so the core logic doesn’t “reach out” to the outside world.

### `GameState`

**Intent:** allow core logic to read/write the current `World` without hard-coding a specific storage mechanism.

Interface:

- `get(): World`
- `put(w: World): Unit`

Handler (`withGameState` in `runtime.effekt`) stores the world in a `Ref[World]`.

### `Rng`

**Intent:** deterministic random numbers.

Interface:

- `nextInt(bound: Int): Int`

Handler (`withRng`) is a small LCG (linear congruential generator) over a `Ref[Int]` seed.
Because the seed lives in a `Ref`, RNG calls naturally “advance” the stream.

### `Log`

**Intent:** store short messages produced by game events (attacks, kills, bumps into walls, etc.).

Interface:

- `write(msg: String)`
- `snapshot(): List[String]`
- `clear(): Unit`

Handler (`withLogBuffer`) keeps a bounded list (newest-first). The main loop calls `Log.clear()`
once per turn so messages don’t stack forever.

### `UI`

**Intent:** rendering and input as an abstract interface.

Interface:

- `draw(w: World, logs: List[String])`
- `read(): String`

Handler (`withTerminalUI`) prints the UI and reads a line from stdin. It uses ANSI escape sequences
to clear the screen between frames.

### `Controller`

**Intent:** “who decides an action?”

Interface:

- `choose(actor: Actor, w: World): Action`

Handler (`withController`) does two things:

- For `PlayerActor`: reads input via `UI.read()`, parses it, and writes an error message to `Log` if invalid.
- For `MonsterActor`: runs the monster AI decision function (`decideMonster`) which uses `Rng` for wandering.

## Execution flow

1. `src/main.effekt` calls `runtime::runGame()`.
2. `runtime::runGame()` runs `play()` inside `console { ... }`.
3. `play()`:
   - reads seed (`readSeed`) and difficulty (`readDifficulty`),
   - creates `seed: Ref[Int]` and `buf: Ref[List[String]]`,
   - creates the initial world with `initWorld(diff)` under `withRng(seed)`.
4. Then it installs handlers (roughly “outside → inside”):

   ```
   withGameState(worldRef) {
     withRng(seed) {
       withLogBuffer(buf) {
         withTerminalUI {
           withController {
             loopGame(initialWorld, diff)
           }
         }
       }
     }
   }
   ```

5. `loopGame` is the main game loop:
   - `snapshot` logs, then `clear` logs (so each turn starts clean),
   - `draw` the world and the last turn’s messages,
   - if `Playing`:
     - get the player action (`Controller.choose(PlayerActor, w)`),
     - apply it (`applyPlayerAction`),
     - if still `Playing`, run all monsters (`monstersTurn`),
     - increment the turn counter and repeat.
   - if `Won` or `Lost`, show an end prompt:
     - `r` resets the world to the stored `init` (retry same level),
     - `n` generates a fresh level via `initWorld(diff)` (using the ongoing RNG stream),
     - `q` quits.

## Dungeon generation and solvability

`generateDungeon` uses a classic “rooms + corridors” approach:

- Start with a fully walled grid.
- Try to place up to `N` random rooms (reject if overlapping).
- Connect room centers in order with an L-shaped corridor chain.
- Place the **start** at the first room center and the **exit** at the last room center (fallbacks exist if no rooms).

Because every room is connected to the next by a corridor, all rooms end up connected in a single component,
so start→exit is reachable in normal cases. The test suite includes a BFS reachability test to guard this invariant.

## Monster AI

Monster decisions are in `logic.effekt`:

- If the monster is adjacent to the player, it “moves into” the player to attack.
- Otherwise:
  - If within an aggro range, it takes a greedy step that decreases Manhattan distance (if the tile is free).
  - If not aggroed, it tries a random free neighbor (wandering).

Difficulty affects the aggro ranges and damage numbers.

## Testing strategy

`src/test.effekt` contains a tiny test harness (no external framework):

- tests are plain computations returning `Bool`
- the runner prints pass/fail lines and exits with a summary

Important tests include:

- input parsing (`parseAction`)
- RNG determinism (same seed → same start/exit)
- “exactly one exit and it is reachable”
- valid monster spawns (on floor, not on the player, no overlaps)
- behavior difference between goblin and rat aggro
- win/quit transitions and log messages

### Adding a new test

- Write a `def test_xxx(): Bool = { ... }`.
- Add it to the `tests` list with a descriptive name.
- Prefer deterministic tests by running under `withRng(ref(seed))`.

## Extending the game

### Add a new monster type

1. Extend `MonsterKind` in `model.effekt`.
2. Update:
   - `kindName`, `monsterChar` in `render.effekt`
   - `monsterHp`, `monsterDamage`, aggro function(s), and `decideMonster` in `logic.effekt`
   - `spawnMonsters` to include the new kind
3. Add/adjust tests to keep invariants (unique positions, valid tiles).

### Change input keys

- Update `parseAction` and the key hint string in `renderStats` (both in `render.effekt`).

### Add new tiles / interactables

- Extend `Tile` in `model.effekt`.
- Update `isWalkable` / `tileChar` and any rules in `logic.effekt`.
- Consider adding tests for the new rule.

