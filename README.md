# VoxelCraft

A Minecraft Classic/Alpha-style voxel sandbox in **one self-contained HTML file**. It uses
Three.js r170 (ES modules plus `PointerLockControls`) and Tailwind CSS v4 (browser build),
both loaded from the jsDelivr CDN. The engine is vanilla JavaScript. Textures, icons and
sounds are generated at runtime, so there are no asset files.

## Run it

Open `index.html` in a current browser (Chrome, Edge, Firefox or Safari with WebGL 2).
It works from the file system or any static server. With GitHub Pages enabled on this repo
it's playable at **https://felixhofmeister1.github.io/Minecraft/**. Touch devices are detected automatically and
get on-screen controls; you can override this in **Settings → Controls**. The world
auto-saves to `localStorage` (on pause, every 30 s and when the tab is hidden), and
**Continue World** on the title screen resumes it.

## Controls

| Action        | PC                                   | Mobile (touch)                                     |
|---------------|--------------------------------------|----------------------------------------------------|
| Move          | `W A S D` / arrows                   | Floating joystick (left side)                      |
| Look          | Mouse (pointer lock, raw input)      | Drag on the right side                             |
| Jump / swim   | `Space` (hold to keep jumping)       | ⤒ button                                           |
| Sneak         | `Shift` (no walking off edges)       | ⇊ toggle button                                    |
| Sprint        | `Ctrl` or double-tap `W`             | Push the joystick all the way forward              |
| Break         | Hold left click (crack animation)    | Break mode: hold still on the right side, or hold ⛏ |
| Place         | Right click (hold to repeat)         | Place mode: tap the right side, or tap ▣           |
| Pick block    | Middle click                         | —                                                  |
| Hotbar        | `1`–`9`, mouse wheel                 | Tap a slot                                         |
| Inventory     | `E`                                  | ▦ button                                           |
| Fly (creative)| Double-tap `Space` or `F`            | Double-tap ⤒ or the wing button                    |
| Pause         | `Esc`                                | ❚❚ button                                          |
| Debug overlay | `F3` (`F1` hides the HUD)            | —                                                  |

**Survival**: blocks are finite, harder blocks take longer to mine, and broken blocks go into
your inventory (grass drops dirt, stone drops cobblestone, glass drops nothing). **Creative**:
infinite blocks, instant breaking, flying, and a block palette in the inventory.

## How it works

| Class / part        | What it does |
|---------------------|--------------|
| Block registry      | IDs 0–7 (Air, Grass, Dirt, Stone, Oak Wood, Oak Leaves, Sand, Bedrock) plus Planks, Cobblestone, Glass, Bricks and Water. Each block defines collision, transparency, render type, sky-light filtering, hardness, per-face textures, sound and drop. The hot loops read flat `Uint8Array` lookup tables. |
| `TerrainGenerator`  | Seeded simplex noise. 2D continentalness, hills, ridged mountains, temperature and humidity give oceans, beaches, plains, forests, deserts and mountains. 3D noise adds mountain overhangs and "spaghetti" caves, sampled on a 4×4×4 lattice and interpolated. It layers grass or sand over dirt over stone, adds a jagged bedrock floor at Y=0–4, water to sea level, and oak trees that are consistent across chunk borders. |
| `Chunk` / `World`   | 16×256×16 chunks stored as `Uint8Array` (`y<<8 | z<<4 | x`) plus per-block vertical sky light. `World` streams chunks around the player, prioritises what's in front of the camera, unloads far chunks, handles block edits (instant re-mesh of touched sections, async re-mesh of light-affected neighbours), DDA raycasts and save data. |
| Web Worker pool     | Terrain generation and meshing run in 1–4 workers built from the same inline script; it falls back to the main thread if workers are unavailable. |
| `Mesher`            | Per 16³ section: floods sky light sideways and downward, then culls hidden faces, computes 4-corner ambient occlusion and smooth light, and greedily merges equal faces into large quads. Vertices are 8 bytes, textures come from a `DataArrayTexture` (repeat-wrapped, mipmapped), and every mesh shares one index buffer. Leaves have Fancy (see-through) and Fast modes. |
| `Renderer`          | Three.js WebGL renderer with custom block shaders (sky light × daylight, AO, fog, animated water). It does its own distance and frustum culling per chunk column and per section, and draws a gradient sky, sun, moon, stars, scrolling clouds, a day/night cycle, the block outline, break cracks and the first-person hand. It also has dynamic resolution. |
| `Physics`           | Player AABB 0.6×1.8, fixed 120 Hz sub-steps with render interpolation, swept per-axis collision (Y, then X, then Z) and a smoothed 1-block auto step-up. Gravity, air drag and a terminal velocity use frame-rate-independent exponential damping. It also covers sneak edge-guard, swimming and creative flight. |
| `InputManager`      | `PointerLockControls` for mouse look, plus keyboard, a floating joystick, a touch look zone and the action buttons, all feeding one set of normalised inputs. |
| `UI`                | Tailwind-styled title, loading, pause, settings and inventory screens (stack, split and shift-move like Minecraft), hotbar, toasts and the F3 overlay. |

Default render distance is 4 chunks on touch devices and 8 on PC (adjustable from 2 to 16).
Touch devices also default to Fast leaves, no MSAA, a capped pixel ratio and dynamic resolution.
The game's core layout CSS doesn't depend on Tailwind, so it stays playable if that CDN fails.
