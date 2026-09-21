# PBR Materials — Metallic-Roughness Discipline

> Source of truth for textures and material presets. Aligned with the Seele
> bridge `MATERIAL_PRESETS` and glTF `pbrMetallicRoughness`.

## Map specifications (glTF metallic-roughness)

| Channel | Map | Gamut | Contents |
|---|---|---|---|
| Base Color | albedo | sRGB | diffuse color (no lighting baked) |
| Normal | normal | linear | tangent-space normals, +Y up (green dominant) |
| Metallic | metallic | linear | 0 = dielectric, 1 = metal; near-binary |
| Roughness | roughness | linear | 0 = mirror, 1 = matte |
| AO | occlusion | linear | contact shadows; blend subtly into base color |
| Emission | emissive | linear→HDR | pure black except emissive areas |
| Alpha | opacity | — | Blend / Cutoff modes only where needed |

- Metallic is near-binary in theory; mid values only for dusty/corroded metal.
- Multiply normal strength by intent: subtle 0.5–1.0 game standard; keep
  micro-detail via noise/grunge overlay in roughness+normal or surfaces read flat.

## Blender node wiring (Principled BSDF)

Base Color → Base Color · Roughness → Roughness · Metallic → Metallic ·
Normal map **through a Normal Map node** → Normal. Standard PBR workflow,
works in EEVEE and Cycles, exports cleanly to glTF.

## Seele material presets (extended for the whole pipeline)

Full parameter table in `data/material-presets.json`; summary:
| Style | roughness | metallic | specular | palette cue |
|---|---|---|---|---|
| organic | 0.80 | 0.00 | 0.30 | living, weathered |
| cyber | 0.20 | 0.90 | 0.80 | chrome, emissive trim |
| mythic | 0.40 | 0.30 | 0.60 | gilded, enchanted |
| dystopic | 0.90 | 0.10 | 0.10 | grime, decay |
| geometric | 0.50 | 0.50 | 0.50 | abstract, clean |

## Texel density (the quality floor)

- Consistent texel density across an asset family beats UV-space efficiency:
  a 1 m area should use ~the same texels everywhere on the ship.
- Scale islands by visual importance: face > back of head; blade ≈ 40% of a
  weapon's UV space; margins 2–4 px at 2048.
- Texture resolution by role: hero 2048, support prop 1024, background 512/256,
  world block trim 1024, terrain megas 4096.
- Always power-of-two dimensions.

## Material count budgets

- 1 material per prop, 1–3 per hero asset, one material set per modular kit.
- Every material is a draw call: atlas textures to pack islands into one sheet
  per kit; remove unused material slots before export.

## Family consistency (AI-assisted texturing)

- One reusable style prompt per asset family: palette, age, edge wear, dirt,
  ornament density, realism, material response — paired with one approved
  reference sheet (ruins, weapons, foliage, mercs…).
- Localized correction: fix one seam/decal/region via inpainting instead of
  regenerating an approved set.
- AI-vertex-color sources: **rebake** — never ship vertex colors as albedo.