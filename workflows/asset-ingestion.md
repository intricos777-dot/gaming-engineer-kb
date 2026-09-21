# Asset Ingestion — Names, Metadata, Consolidation

> Source of truth for how assets enter the remakes' repos and the unified
> manifest. Applies to `worldbuild_manifest.json`, Seele outputs, and the
> dot-hack-remake 1,842-asset consolidation (`t_76319d23`).

## Naming conventions

| Kind | Pattern | Example |
|---|---|---|
| Mesh/GLB | `{type}_{name}.glb` | `char_haseo.glb`, `structure_tower_01.glb` |
| Texture | `T_{Asset}_{Map}.{ext}` | `T_Longsword_Normal.png` |
| LOD | `{name}_LOD{n}.glb` | `char_haseo_LOD2.glb` |
| World block | `{flavor}_{kit}_{cell}.glb` | `cyber_plaza_c04` |
| Zone/world JSON | `zone_{zone}.json`, `world_{game}.json` | `zone_hub.json` |

- No Blender defaults (`Bone.001`, `Cube`); no spaces; snake_case.

## Metadata record — one line per asset (CHARS.md / WORLDS.md / manifest)

```
asset | source (seele seed) | style | budget row | rig family/morphs | QA status | license/notes
```

## Directory contract

```
assets/raw/            AI/seele exports, DCC sources (committed; .blend only < 5 MB)
assets/optimized/      pipeline output — GITIGNORED, rebuilt by CI/task each build
assets/blender/        final committed runtime assets (GLB + textures per workflow)
Content/Data/           game data JSON (zones, canon, chars) per game
```

## Ingestion flow (worldbuild_manifest-driven)

1. Refresh `assets/worldbuild_manifest.json` + `ASSET_INDEX.md`
   (schema: `{generated_at, schema_version, projects, games, summary}`).
2. For each project: locate content root, audit tracked vs on-disk,
   consolidate missing trees (dot-hack-remake: 1,842 assets on disk per manifest
   — seele/cmn_a + vol1_i textures, 80 voice lines, world/zone JSONs, canon
   timeline). Use `git-lfs` for large texture/audio sets.
3. Commit **sources**; keep `assets/optimized/` out of VCS.
4. Record per-asset metadata lines; validator runs in CI/task.

## QA at ingest

- No secrets: scan for tokens/keys before commit (tirith or equivalent).
- No .blend > 5 MB; no `assets/optimized/` balls-of-mud; no duplicate assets
  (dedup by hash) without an intentional reason.
- License/source recorded — mandatory for anything not authored by us.