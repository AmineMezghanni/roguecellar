# Project Name: RogueCellar

RogueCellar is a terminal-based roguelike game. The player explores a randomly generated dungeon, fights monsters, and tries to reach the exit without dying. The game is turn-based: the player acts first, then all monsters take their actions.

## Must-have

Things I want to implement:

- Text-based dungeon map  
  - Display a grid (walls, floor, player, monsters, exit) using ASCII.  
  - Player can move around, but not walk through walls.

- Random dungeon generation  
  - Create simple dungeons with rooms and corridors.  
  - Place the player, monsters, and an exit so the level is solvable.

- Turn-based loop  
  - One action per player turn.  
  - After the player moves or waits, all monsters take a turn.

- Monsters with AI  
  - At least two different monster types.  
  - Monsters react to the player (e.g. move towards them when in range).  
  - When next to the player, they attack.

- Combat and game over  
  - Player and monsters have HP.  
  - Attacks deal damage, monsters and the player can die.  
  - Clear win condition (reach exit) and lose condition (player dies).

- Basic input handling  
  - Simple keyboard commands entered line-based in the terminal (e.g. `w/a/s/d`, wait, quit).  

## Can-have

Things I’d like to add if there is enough time:

- Field of view / “fog of war” (only show what the player can see).
- Monster AI improvements:
  - Pathfinding: instead of just “step in direction of player”, compute a shortest path on the dungeon grid (e.g. A*).
  - Different behaviours per monster type, for example:
    - melee monster: move next to you and attack  
    - ranged monster: try to stay at distance 2–3  
    - cowardly monster: run away when low HP  
  - A simple evaluation of moves: pick the move that reduces distance to the player or avoids obviously bad positions.
- Progression:
  - Items like potions or a slightly better weapon.
- Multiple dungeon levels (e.g. stairs to a deeper floor).

## Will-not-have

Things I do not plan to include:

- GUI.
- Networked multiplayer or online features.
- Complex story, quests, or dialogue system.
- Big RPG systems (no large skill trees, no complex item system).
- Long persistent campaigns with save/load; focus is on short runs.

## Effects and Handlers

Rough plan for effects and how I want to use them:

- `GameState`  
  - Read and update the game world:
    - player position and HP  
    - monsters and their positions  
    - tile information (wall/floor/exit)  
  - Main handler: in-memory state for the running game.

- `Rng`  
  - Random dungeon generation and random choices in AI.  
  - Handler using standard library randomness.

- `UI`  
  - Operations for drawing the current state (map + status) and reading the next player command.  
  - Implemented using terminal input/output.

- `MonsterAI`  
  - Decide each monster’s next action based on the current world.  

- `Log`  
  - Append messages like “The goblin hits you for 2 damage”.  
  - Handler that prints logs or ignores them.

I also plan to structure player input and monster behaviour through a similar effect-based “controller” interface, so that both the human player and the AI-controlled monsters are handled in a uniform way.

## FFI and Libraries

I plan to use only the Effekt standard library:

- Data structures  
  - `list`, `option`, possibly `map` for storing monsters, positions, and dungeon tiles.

- Randomness  
  - Standard library random support, used through the `Rng` effect.

- Console / IO  
  - Standard input/output for reading line-based commands and printing the map.  
  - The `tty` module for nicer terminal output (e.g. clearing/redrawing or simple highlighting), as far as it is useful for this project.
