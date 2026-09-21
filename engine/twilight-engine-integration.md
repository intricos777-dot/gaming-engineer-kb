# Twilight Engine Integration — Wiring the KB into Engine & Loader

> How the library maps onto the Twilight engine modules and the Seele loader.
> Engine changes land in the canonical repo (`twilight-elysium`) and vendor
> outward — one engine task in flight at a time (all touch CMakeLists).

## Current engine state (2026-09-21)

- Stub modules awaiting implementation: `src/asset/asset_stub.{h,cpp}`,
  `src/pipeline/frame_graph_stub.{h,cpp}`, `src/shader/shader_stub.{h,cpp}`
  (board tasks `t_71510adf`, `t_54d948f6`, `t_7353d59d`).
- Loader exists: `src/ai/blender_asset_loader.{h,cpp}` — `BlenderModel`
  (name, path, format, vert/face counts, vao/vbo/ebo, uploaded flag) and
  `PBRTextureSet` (albedo/normal/roughness/metallic paths).

## Implementation mapping (KB → code)

### Asset module (`asset_stub` → real)
- Consume `BlenderModel` + `PBRTextureSet` per `core/04` + `core/05`.
- Import path: `.glb` (glTF binary) primary; `OBJ`/`FBX` legacy fallback only.
- Texture upload: PNG/JPEG for compat; add KTX2 (Basis) decode path per
  `data/compression-matrix.json` — ETC1S for color/ORM, UASTC for normal.
- Mesh upload: positions/normals/UVs/joints/weights as typed arrays
  (float32 pos, uint16 indices) direct to GPU (Vulkan-style buffer layout).

### Frame graph module (`frame_graph_stub` → real)
- Nodes/passes per `core/02` streaming hierarchy: ground → detail → props,
  per-cell submit; culling by cell bounds; LOD switch by distance
  (budgets `data/lod-budget-tiers.json`).
- Resource transitions for upload → render; interleave streaming loads
  (async cell in, hysteresis unload) per `core/02 §1`.

### Shader module (`shader_stub` → real)
- Compile/cache from `assets/shaders/`; PBR metallic-roughness shader per
  `core/04`; skinned mesh path per `core/03 §2` (precomputed `J*IB` palette,
  4-influence LBS in vertex stage); morph targets via `WEIGHTS` channels with
  sparse activation (`core/03 §4`).
- Handle the missing-meshopt fact: no `EXT_meshopt_compression` dependency.

## Seele→engine contract (already running)

```
Seele (Blender headless) ──GLB──▶ blender_asset_loader ──GPU structs──▶ frame graph
        └── seele_manifest.json ──▶ asset registry (name→model/textures)
```

- Manifest fields: asset name, style preset, seed, format, paths.
- Deterministic rebuild: same seed → same world (rule in `core/06`).

## Vendoring (after engine modules land)

Copy completed modules to: black-ops-2-elysium, tf2-elysium, half-life-elysium,
half-life-2-elysium (`tools/twilight-elysium/src/...`), loz-majoras-mask
(`third_party/te/src/...`). Preserve path layout; verify each repo still
configures via CMake (board task `t_494bb7f5` then `t_9e33e90d`).

## Acceptance for this phase

- `t_cb375af8`: an exported Seele `.glb` loads headless through the loader,
  uploads to GPU structs, renders one test frame or logs success.
- Alpha milestone (`t_225de320`): one engine test scene with representative
  slice of the 2,121-asset manifest + new Seele GLBs, rendering within budget.