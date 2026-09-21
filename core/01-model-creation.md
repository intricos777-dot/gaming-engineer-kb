# 3D Model Creation — AI-Mesh Era Pipeline

> Source of truth for turning any generated mesh (Seele procedural, Tripo/Meshy/Rodin
> AI output, photogrammetry, or hand-built) into a game-ready asset.

## Pipeline (order is binding)

**1. Concept brief → reference image** (AI-assisted)
- Prompt as production brief: object + purpose + style + material + scale +
  constraints. Example: *"low-poly medieval market stall, game prop, wood and
  cloth, clean silhouette, no text, no background, under 5k triangles."*
- Image-to-3D beats text-to-3D: generate a clean front-on reference first; the
  3D step reconstructs far more reliably from an image.
- Generate **3–8 drafts**, choose on silhouette + readability + repairability.

**2. Generate base mesh** (Seele bridge or AI service)
- Expect raw AI output problems: 50k–500k tris for a prop, non-manifold faces,
  internal geometry, zero-area triangles, disconnected verts, no usable UVs,
  vertex-color-only materials. **This is normal; never ship it raw.**

**3. Cleanup** (Blender, always first)
- `Mesh > Clean Up > Degenerate Dissolve`, `Delete Loose`; merge by distance.
- Recalculate normals (`Shift+N`); check flipped faces (Face Orientation overlay);
  3D Print Toolbox identifies non-manifold edges.
- Decimate only with **Un-Subdivide/Planar** (never Collapse at this stage).
- Apply scale/orientation: `Ctrl+A > All Transforms`; 1 unit = 1 m.

**4. Retopology** — the topology gate (non-negotiable for deforming meshes)
| Method | Use for | Cost |
|---|---|---|
| Manual (Poly Build + Shrinkwrap) | Hero characters, weapons, deformable | 30–60 min prop; 2–4 h character |
| QuadriFlow Remesh | Rigid props, batch | fast, poor edge flow for deform |
| Instant Meshes | Organic shapes | better flow than QuadriFlow |
| Voxel Remesh (intermediate) | clean manifold before QuadriFlow | fast |
- Edge loops follow contours; quads where possible; density concentrates at
  detail (eyes, mouth, joints).
- Static props on UE5-class pipelines may skip retopo (Nanite) — not our engine;
  **our engine requires explicit LODs.**

**5. UV mapping**
- Hard-surface props: Smart UV Project, angle limit 66–72°.
- Seams in hidden areas (inner legs, underarms, material boundaries); check
  stretching immediately.
- Sail: islands packed with 2–4 px margin, scale by visual importance, consistent
  orientation (+V up), **texel density consistency** across the asset
  (Texel Density Checker).

**6. PBR material rebake** (see `core/04-pbr-materials.md`)
- New material per channel: Base Color, Normal, Roughness, Metallic, AO.
- Bake high-poly (or the AI mesh) → low-poly retopologized target; normals in
  tangent space; AO with raised ray distance; inflate high-poly slightly to avoid
  halo artifacts.
- Post: levels on roughness, seam cleanup on normal edges, manual paint where AI
  textures artifact. Material IDs (skin/metal/cloth masks) from AI segmentation,
  refined by hand.

**7. LOD chain** (see `data/lod-budget-tiers.json`)
- LOD0 = retopologized target. LOD1 = 50%, LOD2 = 25%, LOD3 = 10% (silhouette-first
  simplification, billboard/impostor at LOD3 where allowed).
- Preserve UVs between LODs; re-bake textures when UVs change.

**8. QA gates — nothing enters an engine without these**
- Non-manifold geometry: 0 results (Select → All by Trait → Non-manifold).
- No flipped normals; no loose geometry.
- UV islands: no overlap (except intentional tiling); no islands outside 0–1.
- Triangle count within `data/lod-budget-tiers.json` for the asset tier.
- Power-of-two textures; textures sized to final viewing distance (hero 2k,
  background 256–512).
- Pivot/logical origin set; scale applied; no hidden or disabled objects exported.
- Bone names descriptive (no `Bone.001` defaults) — see `core/03-animation.md`.
- **Time rule:** fixing a bad base costs more than regenerating. If topology is
  chaotic AND UVs unreadable AND shape unclear → regenerate, don't repair.

## Automation guidance for agents
- Automate mechanical stages (cleanup, decimation, LODs, export) headlessly via
  bpy; keep judgment (seams, retopo decisions, style review) as explicit gates.
- Batch scripts: input dir of raw meshes → cleanup → remesh at target count →
  auto-UV → bake from vertex colors → LODs → export. Produce a per-asset QA
  report as JSON the task can verify.
- Semi-automation is the studio sweet spot: machines do the repetitive passes,
  the agent/artist reviews the creative gates.

## References
StraySpark (2026-04) Blender-to-engine pipeline; Tripo (2026-07) studio asset
pipeline & AI workflow; aivyber.cz (2026-06) AI 3D workflow; Sculptor (2026-08)
AI 3D workflow consensus.