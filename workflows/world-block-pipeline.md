# World-Block Pipeline — Seele Batch → Assembly → Streaming Cell

> Applies `core/02` + `core/06` to worlds. Run per game flavor; each world block
> is a **cell-scale kit** on a fixed lattice.

## Stages

1. **Flavor palette** — material style per game (cyber/mythic/dystopic/organic);
   lock the kit's style prompt + reference sheet (family consistency rule).
2. **World block batch** — Seele bridge, `--model-type structure`, fixed
   `--seed`, output `~/.twilight-elys/blender_export/`. 3 blocks per flavor per
   batch (task `t_6dbe4e8b`). Update `seele_manifest.json`.
3. **Kit discipline** — every block connects on the lattice (repeatable
   spacing); seams hidden by design; one material set per kit (atlas).
   Budget row: `data/world-block-sizes.json`.
4. **Assembly** — zone + world JSON per game from the unified manifest
   (`assets/worldbuild_manifest.json`); place blocks in cells; deterministic
   scatter (same seed → same world).
5. **Streaming config** — cell sizes/radii per `data/world-block-sizes.json`
   (terrain 1000 / detail 500 / props 250); hysteresis on unload; priority
   ground→detail→props; preload spawn cell.
6. **Optimize** — GLB + Draco + KTX2 at build time; per-cell kit under Excel
   budget (world block < 20 MB).
7. **Integrate + verify** — load through blender_asset_loader into the engine
   test scene; render 1 frame; report fps/VRAM + draw calls vs budget.

## Flavor → block palette (binding from coordination contract)

| Flavor | Kit | Games |
|---|---|---|
| cyber | plazas, towers, transit | black-ops-2-elysium, tf2-elysium |
| mythic | ruins, sanctums, chimeric | twilight-elysium, kingdom-hearts-zero, loz-ocarina-mq |
| dystopic | City17/Black Mesa industrial | half-life pair, Elder Scrolls pair |
| organic | living terrain, bio-structures | ringworld-redux, te-bonfire, te-halo-inspired |

## Acceptance

- Blocks tile seamlessly on the lattice at origin; no normal/UV seams visible.
- Deterministic: rebuild from seed reproduces identical cell placement.
- Loads + frames in the engine test scene within budget.