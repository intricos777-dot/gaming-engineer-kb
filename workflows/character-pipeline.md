# Character Pipeline — End to End (for the remakes)

> Applies `core/01`–`core/07` to characters. Run per character; record results
> in the board task.

## Stages

1. **Brief** — flavor per game (material style from coordination contract):
   cyber (bo2, tf2) · mythic (twilight, KH-zero, oot-mq) · dystopic (HL pair,
   ES pair) · organic (ringworld, te-bonfire, te-halo).
2. **Generate** — Seele bridge batch, `--model-type character`, fixed `--seed`,
   output `~/.twilight-elys/blender_export_chars/`. Update `seele_manifest.json`.
3. **Repair/topology gate** — cleanup (degens, loose, normals), retopo to
   budget tier for a character (LOD0 30–60k), edge flow at joints.
4. **UVs** — seams hidden; texel density consistent; 4 px margins at 2048.
5. **PBR materials** — style preset + per-material masks; bake AO → blend;
   rebake any AI/vertex-color source.
6. **Rig** — canonical skeleton per family; deform bones only; weights
   normalized, Limit Total 4; **export from rest pose**; sockets for props
   (hands/back/hips, documented orientation).
7. **Morph targets** — expression set per `data/animation-сlip-presets.json`
   (blink, mouth phonemes, brow) as additive shape keys; sparse-optimized.
8. **LODs** — 100/50/25/10; UV-preserving.
9. **Export** — GLB, canonical settings; **Draco+KTX2 via gltf-transform at
   build time**, not in the repo.
10. **QA gates** — validator 0 errors; triangle/texture/draw-call budgets;
    in-engine frame + joint movement test (deformation check with a validation
    pose set: T-pose, 90° arm raise, full crouch, twist).

## Artifacts

```
assets/blender/chars/<name>/        # committed sources (GLB + textures, no .blend > 5MB)
assets/optimized/chars/<name>/*.glb # build output (gitignored)
CHARS.md line per character: seed, style, budget row, rig family, morph set, QA status
```

## Acceptance

- Loads through `blender_asset_loader` (no GPU errors, texture sets attached).
- Deforms without volume collapse or candy-wrapper on validation poses.
- Within `data/lod-budget-tiers.json` row for hero characters.