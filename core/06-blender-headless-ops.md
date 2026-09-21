# Blender Headless Operations — The Forge the Guild Shares

> Source of truth for running Blender without a UI ("the forge") from Seele,
> Hermes workers, or opencode. Verified against **Blender 5.2.2 LTS** at
> `/usr/bin/blender` (Garuda/Arch, 2026-09-21).

## Invocation patterns

```bash
# expression mode (tiny checks)
blender --background --python-expr "import bpy; print(bpy.app.version_string)"

# script mode (production batches)
blender --background --python script.py -- --arg value

# Seele bridge (already proven: exit 0, exports land in ~/.twilight-elys/)
blender --background --python tools/blender_seele_bridge.py -- \
  --export-models --model-type character --format glTF \
  --output-dir ~/.twilight-elys/blender_export_chars --seed 20260921
```

## Seele bridge CLI (the shared forge API)

```
--export-models          export all models
--output-dir PATH        destination (default ~/.twilight-elys/blender_export)
--format {glTF,OBJ,FBX}  always glTF for engine delivery
--model-type             all | tree | character | monster | weapon | structure
--generate-material      build material sets
--material-style         organic | cyber | mythic | dystopic | geometric
--texture-size N         texture resolution cap
--seed N                 determinism seed — ALWAYS set
```

## bpy patterns every agent needs

- Clean scene first: `bpy.ops.object.select_all(action='SELECT'); bpy.ops.object.delete()`.
- Procedural generators: `bpy.ops.mesh.primitive_*_add()`, then modifiers —
  apply modifiers before export (`bpy.ops.object.modifier_apply`).
- Materials: create Principled BSDF, wire Image Texture nodes, or drive
  `node_group` for presets — see `core/04-pbr-materials.md` wiring.
- Export: `bpy.ops.export_scene.gltf()` with the canonical settings in
  `data/gltf-export-settings.json` (apply as kwargs).
- LOD batches: decimate copies at ratio 0.5 / 0.25 / 0.1, export each
  `name_LOD{n}.glb` with shared UVs.
- **Always** wrap batches in `try/except` and write a JSON report
  (`{asset, status, triangles, materials, files[]}`) — tasks verify on the
  report, not on silence.

## Determinism rule

- Same `--seed` + same script + same Blender version → identical output.
- Record seed in the task/board and in asset metadata
  (`data/asset-ingestion` convention).
- Random generation without a seed is a defect — scatter/decor should be
  deterministic per world cell (see `core/02-world-creation.md`).

## Environment notes (Blender 5.2.2 LTS here)

- `libbf_intern_meshopt_bridge.so` missing means **MeshOptimizer compression is
  unavailable in the glTF exporter** — exports still succeed; the warning is
  benign. Do not depend on the `EXT_meshopt_compression` extension until the
  lib is installed.
- `bpy` is importable **only inside** Blender's interpreter, not system python —
  run everything through `blender --background --python`.
- Batch timeouts: model batches ≤ 180 s per pass at seed scale used so far;
  raise for heavy `structure` batches; never let a batch run unbounded.

## References
Blender 5.2 docs (headless/bpy); StraySpark automation guidance (2026);
verified on this machine 2026-09-21 (character×4 + structure×5 batches,
exit 0, manifest updated).