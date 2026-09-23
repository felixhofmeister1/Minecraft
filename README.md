# VoxelCraft

A Minecraft (Java Edition) remake in **one self-contained HTML file**: Survival, Hardcore and
Creative, infinite procedural worlds, the Nether and the End, 67 mobs including the Ender
Dragon and the Wither, and Java-style mechanics throughout. It runs on PC and on phones.

It uses Three.js r170 and Tailwind CSS v4 (browser build), both from the jsDelivr CDN.
Everything else is generated at runtime: textures, item icons, mob models, sounds and music.
There are no asset files.

## Play

Open `index.html` in a current browser with WebGL 2 (Chrome, Edge, Firefox or Safari). It
works straight from the file system or from any static server. It's also live at
**https://felixhofmeister1.github.io/Minecraft/**.

Worlds are saved in the browser (IndexedDB): every minute, on pause, and when the tab is
hidden. **Singleplayer** lists them and **Create New World** starts one. When you create a
world you pick the mode (Survival, Hardcore or Creative), the difficulty, whether cheats
are allowed, and optionally a seed.

## Controls

The game detects touch devices and shows the on-screen controls automatically. You can
override this in **Options → Controls**.

| Action | PC | Mobile |
|---|---|---|
| Move | `W A S D` | Floating joystick (left half of the screen) |
| Look | Mouse (pointer lock, raw input) | Drag on the right half |
| Jump / swim up | `Space` | ⤒ |
| Sneak | `Shift` | ⇊ (toggle) |
| Sprint | `Ctrl` or double-tap `W` | Push the joystick all the way |
| Attack / mine | Left click (hold to mine) | Big action button in **Break** mode, or hold on the look area |
| Use / place | Right click (hold to repeat) | Big action button in **Place** mode, or tap the look area |
| Break ⇄ Place mode | — | **MODE** button |
| Interact (open, eat, draw bow, raise shield…) | Right click | **USE** button (hold) |
| Pick block | Middle click | — |
| Drop item / drop stack | `Q` / `Ctrl+Q` | **DROP** (tap / hold) |
| Swap with off-hand | `F` | **SWAP** |
| Inventory | `E` | ▦ |
| Hotbar | `1`–`9`, mouse wheel | Tap a slot |
| Fly (Creative) | Double-tap `Space` | Double-tap ⤒ or the wing button |
| Chat / command | `T` / `/` | 💬 |
| Camera (1st / 3rd person) | `F5` | 👁 |
| Hide HUD / screenshot / debug | `F1` / `F2` / `F3` | — |
| Pause | `Esc` | ❚❚ |

Container screens follow Java click rules:
- **Left click:** pick up or place a stack.
- **Right click:** split a stack, or place one item.
- **Shift-click:** quick move.
- **Double-click:** collect all of that item.
- **Drag:** spread items across slots.
- **`1`–`9`, `F`, `Q`:** swap with a hotbar slot, swap with the off-hand, drop.

On touch screens:
- **Tap:** pick up or place.
- **Long-press:** split.
- **Double-tap:** quick move.

## What's in it

**World and rendering**
- Infinite 16×256×16 chunks. Generation and meshing run in a pool of Web Workers.
- Face culling, smooth lighting with ambient occlusion, and distance LOD.
- 0–15 sky and block light using Java's lightmap curve, with a 20-minute day/night cycle
  and eight moon phases.
- 256 block types: stairs, slabs, fences, walls, doors, panes, crops, redstone parts,
  fluids and more.
- 33 biomes, including Lush Caves, Dripstone Caves and the Deep Dark.
- Weather: rain, snow, thunderstorms and lightning, with snow and ice building up over time.
- Spaghetti tunnels and cheese caverns, ores, and trees for every wood type.
- Overworld structures: villages, desert pyramids, jungle temples, swamp huts, ocean
  monuments, woodland mansions, strongholds (with the End portal), mineshafts, pillager
  outposts, shipwrecks and ruined portals.
- The Nether: fortresses, bastions and all five Nether biomes.
- The End: the obsidian pillars with crystals, the dragon fight, the exit portal and egg,
  End gateways, outer islands, and End cities with ships (and elytra).

**Survival**
- Health, hunger with saturation and exhaustion, oxygen, fall damage, drowning,
  suffocation, fire, lava and the void.
- XP with Java's level curve.
- All 33 status effects.
- Death drops, `keepInventory`, respawning at your bed. In Hardcore, death switches you to
  spectator.

**Combat**
- Attack cooldown, critical hits, sprint knockback, sword sweeps.
- Shields; axes disable them for 5 s.
- Armor and toughness, Protection EPF, the Totem of Undying.
- Bows, crossbows (multishot and piercing), tridents (loyalty, riptide, channeling),
  and splash and lingering potions.

**Crafting and processing**
- 238 recipes in the 2×2 and 3×3 grids, including mirrored shapes and tool repair.
- Furnace, blast furnace and smoker.
- Brewing stand with every Java potion and modifier.
- Enchanting table: all 39 enchantments, bookshelf levels and the Java offer algorithm.
- Anvil with repair, combining, renaming, the prior-work penalty and "Too Expensive!".
- Grindstone and smithing table (netherite upgrades).
- Villager trading with profession levels.

**Redstone**
- Dust with 15 power levels and strong/weak powering; torches with burnout.
- Repeaters (with locking) and comparators (compare/subtract, container reading).
- Levers, buttons, pressure plates, observers, target blocks, sculk sensors, lamps,
  note blocks and TNT.
- Dispensers, droppers and hoppers.
- Pistons and sticky pistons with Java's 12-block push limit.

**Mobs**
- 65 mobs plus the Ender Dragon and the Wither, covering the passive, neutral and hostile
  lists, with their behaviours: breeding, taming, riding, bartering, shearing, milking,
  creepers, Endermen carrying blocks, guardian lasers, phantoms, and the Warden (darkness,
  sonic boom, summoned by shriekers).
- A* pathfinding.
- Java-style spawning: mob caps, light-level rules, per-biome and per-structure spawn tables.

**Audio**
- Spatial (HRTF) sound for blocks, mobs, weather and ambience, with cave reverb and
  generative background music.

**Commands** (with cheats on)
`/gamemode /time /weather /give /tp /kill /summon /effect /enchant /xp /clear /difficulty
/seed /locate /gamerule /spawnpoint /setblock /fill /heal /feed`

## How it's built

`index.html` contains three scripts:

| Script | Purpose |
|---|---|
| `voxel-shared` | The engine core, shared by the page and the workers: block registry, collision shapes, world generation (terrain, biomes, caves, structures, Nether, End), light propagation and the mesher. |
| `voxel-worker` | A small worker entry point. The page builds a Blob worker from it plus the shared core. Generation and meshing run in 1–4 workers, with a main-thread fallback. |
| `type="module"` | The game: Three.js renderer and shaders, entities and physics (20 ticks per second, Java movement rules, render interpolation), mobs and AI, block logic (fluids, fire, growth, explosions), redstone, containers and screens, UI, audio, input and saving. |

## Differences from Java Edition

This is a from-scratch web reimplementation, not a port.
- **Blocks:** limited to 256 types, so less common blocks and variants are missing.
- **Generation:** follows the modern (1.18+) look, but the terrain and structures are this
  game's own layouts, not identical to Java for the same seed.
- **Content not included:** multiplayer, advancements beyond a few chat messages, maps,
  banners, and some late-version content (archaeology, trial chambers, the 1.21 mobs).
