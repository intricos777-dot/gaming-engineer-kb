# 3D Model Animation — Rigging, Skinning, Morph Targets, Retargeting

> Source of truth for characters that deform correctly, share animations, and
> stay inside bone budgets. glTF/GLB is our delivery format; raw math included
> so engine-side debugging is diagnosis, not mystery.

## 1. Skeleton conventions (canonical first)

- One canonical skeleton per character family: bone naming, hierarchy, rest pose
  (T or A), documented in the repo.
- Industry-standard naming (Rigify/Mixamo-compatible) so retargeting works:
  `spine`, `spine_upper`, `neck`, `head` · `thigh_l`, `shin_l`, `foot_l`, `toes_l`.
- **Engine loader rule:** never hardcode joint names — discover from the glTF
  file or a semantic-name config map.
- **Bone budgets:** reduce joint count aggressively (twist joints → corrective
  shapes, collapse finger chains on background characters, strip helper bones).
  Separate control/helper bones from **deformation bones**; export deformation
  bones only unless IK needs references.

## 2. Skinning (LBS by default, DQS only for film)

Skeletal Subspace Deformation / Linear Blend Skinning — the game-industry default:

```
jointMatrix(j) = globalTransformOfJointNode(j) * inverseBindMatrix(j)
skinnedPosition = (w0*J0*IB0 + w1*J1*IB1 + w2*J2*IB2 + w3*J3*IB3) * restPosition
```

- Precompute `J*IB` per joint on the CPU; upload only the joint-matrix palette.
- **4 influences per vertex max** (glTF `JOINTS_0`/`WEIGHTS_0`) — use Blender's
  `Limit Total` (N=4) + `Clean Weights` + **normalize weights** (sum = 1.0)
  before export. Unnormalized weights = shrinking/expanding vertices.
- DQS preserves volume on twist but bulges at joints, mishandles scale, and
  breaks engine compatibility — industry consensus: **avoid DQS in games**; fix
  volume loss with corrective blend shapes.
- Automated binding (heat map / geodesic voxel) then refine the hotspots: hips,
  knees, elbows, shoulders, spine.

## 3. Bind pose discipline (the silent killer)

- All animation data is a **delta from the bind pose**. glTF stores it as the
  **Inverse Bind Matrix** per joint.
- **Export only from rest pose.** A posed-at-export rig produces wrong inverse
  bind matrices and a deformed T-pose in engine.
- Bind pose must match exactly between DCC and engine; joint indices are
  referenced by index — reordering joints after export breaks every animation.

## 4. Morph targets / blend shapes (glTF `WEIGHTS`)

- For faces & fine deformation: store vertex-position deltas per target; runtime
  interpolates linearly — trivially GPU-parallelizable:
  `final = base + Σ weight[i] * displacement[i]`.
- Shape keys in Blender: `Basis` first, semantic names (`blink_l`, `smile`,
  `mouth_open`) exported via `extras.targetNames`; keep targets **additive**.
- **Memory math:** 50 targets × 50k verts × 12 B/vec ≈ **30 MB/character face**
  (60 MB with normals). Mitigate: sparse accessors (glTF) store only changed
  verts (blink = eyelids only), bindless morph buffers + sparse activation
  (promote only active targets to GPU).
- Faces = skeletal joints for gross motion (jaw open) **+ dense morph layer**
  (lip sync, brow, nostril) — the professional two-layer scheme.

## 5. Correctives & IK/FK

- Corrective blend shapes fix loss in extreme poses (arm above 90° shoulder
  collapse; candy-wrapper twist) — pose-space deformers keyed to pose.
- Production limbs: one FK chain + one IK chain + blended output (0–1 blend
  attribute); pole vectors + twist joints stabilize elbow/knee, prevent flips.
- Foot-plant correction (anti foot-skate, plant detection + knee-pop-free
  stretch), grounded-feet stance recalibration, optional arm IK, root motion
  keep/strip/extract.

## 6. Retargeting

- Map source skeletons (Mixamo, ActorCore, UE Mannequin, BVH mocap, Rigify) onto
  our canonical rigs by semantic name/hierarchy; topology fallback for
  meaningless names; geometric solvers (canonical anatomical frames) beat
  direction-copying; natural shoulder/neck/head/foot carriage inherits only the
  source's *motion*, not slumped posture.
- Save confirmed mappings as profiles keyed by skeleton signature — recognition
  is instant and permanent per signature.

## 7. Animation runtime (state machines, not spaghetti)

- Split clips and name them for machines: `idle`, `walk`, `run`, `jump_land`,
  `attack_sword_1`, `hurt_light`, `death` (see `data/animation-clip-presets.json`).
- Blend trees for locomotion (direction + speed); root motion on locomotion,
  extract for attacks unless gameplay needs body travel; loop flags carried in
  metadata.
- Sample FPS standard (e.g. 30/60); `resample` drops redundant keyframes without
  changing playback (gltf-transform).

## 8. glTF export checklist (characters)

```
Format: .glb · Include: Selected Objects, Custom Properties (physics metadata)
Transform: +Y Up · Mesh: UVs, Normals, Tangents, Vertex Weights, Shape Keys
Armature: Use Rest Position Armature, Export Deformation Bones Only (unless IK refs),
          Add Leaf Bones OFF · Animation: Animation ON, Shape Key Animation ON
```

## References
Khronos glTF Tutorials (skins, morph targets) & Vulkan Compute Skinning docs;
industry rigging guides (2026); humanoid-retargeter (s&box) retargeting study.