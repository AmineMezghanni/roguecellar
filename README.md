# RogueCellar

A small terminal roguelike written in **Effekt**.

You explore a dungeon, deal with monsters, and try to reach the exit.

## Features

- **Seeded procedural generation**: same seed → same dungeon layout.
- **Difficulty modes** (Easy / Medium / Hard): changes player HP, monster count, monster HP/damage, and aggro ranges.
- **Two distinct monster behaviors**:
  - **Goblins** chase from farther away.
  - **Rats** only chase when close; otherwise they wander.
- **Retry / New level** after you win or lose: replay the same level (`r`) or generate a new one (`n`).

## Getting started

### Requirements

- An **Effekt** toolchain (the `effekt` command available in your terminal).

### Clone the repo

```bash
git clone https://github.com/AmineMezghanni/roguecellar.git
cd roguecellar
```

### Run the game


```bash
effekt src/main.effekt
```

### Run the tests

```bash
effekt src/test.effekt
```

## How to play

When the game starts:

1. Enter a **seed** (an integer).  
   Using the same seed will reproduce the same dungeon and spawns.
2. Choose a **difficulty**:
   - `e` = Easy
   - `m` = Medium
   - `h` = Hard

### Controls

The default controls are chosen to work on an AZERTY/French keyboard:

- `e` = up
- `s` = left
- `d` = down
- `f` = right
- `.` = wait
- `q` = quit

### Win / lose menu

After you **win** or **lose**, you can choose:

- `r` = retry the same level (same seed + same layout + same spawns)
- `n` = generate a new level (continues using the RNG stream from the initial seed)
- `q` = quit

## Project layout

```
src/
  main.effekt            Entrypoint (calls runtime::runGame)
  test.effekt            Tests / test harness
  rogue/
    model.effekt         Core types + effect interfaces (GameState, Rng, Log, UI, Controller)
    util.effekt          Small list/math helpers
    dungeon.effekt       Dungeon generation + tile utilities
    logic.effekt         Game rules (movement, combat, monster AI, turn loop helpers)
    render.effekt        Rendering (map → strings, stats, parsing input)
    runtime.effekt       Handlers + terminal UI + game bootstrap
```

## Notes

- For a deeper overview of the architecture (especially effects/handlers), see `ARCHITECTURE.md`.
