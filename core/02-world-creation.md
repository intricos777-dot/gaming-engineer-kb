# 3D World Creation — Tile-Based Worlds That Stream

> Source of truth for building worlds that are performant by default:
> partition first, scatter deterministically, stream cells, never unbounded scenes.

## 1. World partition (the ground rule)

Split the world into a spatial grid of **cells**, each with its own cell size,
loading radius, unloading radius, and priority.

- **Hysteresis:** cells load when a source (player/camera) enters the *loading
  radius* and unload only past the *unloading radius* — prevents load/unload
  flicker at boundaries.
- **Priority layers:** ground/terrain loads before detail before props
  (terrain 1000 m cells, detail 500 m, props 250 m is a proven default).
- **Preload** cells around spawn to avoid falling through the world.
- **Floating origin / large-world fix:** beyond ~16 km precision degrades —
  stream around a local origin, rebase as the player moves ("floating point
  system"). A 40 km world behaves like a small scene under this regime.

## 2. Terrain

| Technique | Use when |
|---|---|
| Heightmap + stamps (Gaia-style) | authored, art-directed ground |
| FBM noise + octave subdivision (GPU clipmap/octree) | runtime/vast terrain, overhangs+caves |
| Chunked mesh pipeline (heightmap → array-driven LOD meshes) | ECS/streaming integration |
- **Height-blended texturing** (slope-based bands, triplanar projection kills
  seams), **stochastic shading** (kills texture tiling), macro variation across
  kilometers, wetness/snow-specialized shaders where the world calls for it.
- Terrain LOD is real-time: near-high detail, far-coarse; never static tessellation.

## 3. Vegetation & scattering (deterministic)

- **Same seed → same world**, reproducible on every machine and at runtime.
- Density stops being a luxury: variation in scale/rotation/color turns a
  thousand instances of one tree into a forest — not wallpaper.
- Avoidance rules + paint masks: forests part exactly where a quest hut stands;
  fences/power poles run along splines; spawn along lines/shapes/paths.
- **GPU instancing** for repeated props; **impostors** at extreme distance;
  density fadeout + scaleout instead of hard draw-distance pops.

## 4. Modular kits (world blocks)

- Build the world from **modular blocks** (Seele `structure` batches): each block
  is a self-contained cell-scale kit (see `data/world-block-sizes.json`).
- Rule: every block connects on a fixed grid lattice (repeatable spacing), seams
  hidden by design, one material set per kit where possible (1 material per prop
  target, 1–3 per hero asset).
- Building generator rule: exterior-only shells for skyline, full interiors only
  where walkable — spawn both as ordinary level assets.

## 5. Worlds per remake flavor (from the coordination contract)

| Flavor | Worlds | Seele material style |
|---|---|---|
| cyber | black-ops-2-elysium, tf2-elysium | `cyber` |
| mythic | twilight-elysium, kingdom-hearts-zero, loz-ocarina-mq | `mythic` |
| dystopic | half-life pair, Elder Scrolls pair | `dystopic` |
| organic | ringworld-redux, te-bonfire, te-halo-inspired | `organic` |

## 6. Assembly & lighting

- Lightmap UV2 generation for baked lighting; precompute static lighting;
  dynamic lights only where gameplay demands.
- Occlusion culling + frustum culling by cells; LOD switch by distance, not tag.
- Colliders managed per-cell, exportable headless for server builds.
- Audio: cell-scoped ambient soundscapes follow the same streaming hierarchy.

## References
Procedural Worlds Storm/Gaia (2026), SECTR World Streaming, Unity World
Partition (UE5-inspired cell streaming), World Streamer 2, Infinite Lands
(node-based PCG + Job/Burst), Tessaractic (GPU octree terrain, FBM).