# Gaming Engineer Knowledge Base

A living lexicon of 3D model creation, 3D world creation, and 3D animation —
distilled for every gaming engineer AI in the fleet: **Seele**, **Hermes** workers,
and **opencode**. Research-grounded (2026 state of the art), engine-actionable.

> Read `MANIFEST.json` first. It maps every topic to the documents that contain
> the load-bearing numbers, thresholds, and procedures.

## How to consume this library (for AI agents)

1. **Locate** the topic in `MANIFEST.json` (`topics[]`).
2. **Load** the listed `core/*.md` document.
3. **Apply** the numbered pipeline; never skip the *QA gates*.
4. **Consult** `data/*.json` for the exact presets (materials, LOD tiers, world
   block sizes, compression matrix, export settings) — these are the same
   values the **Seele** Blender bridge and the Twilight engine consume.
5. **Record** decisions and validation output in the task/board before 'done'.

## Layout

| Path | Contents |
|---|---|
| `core/01-model-creation.md` | AI-mesh → cleanup → retopo → UV → PBR bake → LOD → export |
| `core/02-world-creation.md` | World partition, streaming, terrain, scattering, modular kits |
| `core/03-animation.md` | Rigging, skinning math, morph targets, retargeting, state machines |
| `core/04-pbr-materials.md` | Metallic-roughness maps, material presets, texel density |
| `core/05-gltf-pipeline.md` | Draco, KTX2, gltf-transform, engine support matrix, validation |
| `core/06-blender-headless-ops.md` | Headless bpy patterns, batch ops, determinism |
| `core/07-optimization-budgets.md` | Triangle/LOD/material/texture budgets by asset tier |
| `workflows/character-pipeline.md` | End-to-end character asset flow for the remakes |
| `workflows/world-block-pipeline.md` | World block flow: Seele batch → assembly → streaming cell |
| `workflows/asset-ingestion.md` | Name/metadata/ingest conventions (worldbuild_manifest) |
| `engine/twilight-engine-integration.md` | Wiring KB rules into Twilight engine stubs + loader |
| `data/*.json` | Machine-readable presets (see MANIFEST) |

## Canonical rules (the few that bind everything)

- **Scale:** 1 Blender unit = 1 meter; apply transforms before export.
- **Coordinate:** glTF is +Y Up; the engine loads with its own convention — keep
  orientation consistent at export (see `data/gltf-export-settings.json`).
- **Delivery:** GLB + Draco + KTX2 for runtime; keep sources uncompressed.
- **Determinism:** every generative batch uses an explicit seed (see
  `core/06-blender-headless-ops.md`).
- **Engine first:** engine code changes land in `twilight-elysium`, then vendor
  outward — never fork (coordination contract
  `~/.agent-coordination/game-remakes-coordination.md`).

Generated: 2026-09-21 · Research sources: Khronos glTF tutorials & Vulkan docs,
Khronos 3D-on-the-Web texture compression talk, StraySpark & Tripo & aivyber
2026 pipeline guides, gltf-transform/Unity-glTFast/Babylon.js optimization
guides, Procedural Worlds (Storm/Gaia), SECTR/World-Partition streaming docs,
character-rigging industry guides.