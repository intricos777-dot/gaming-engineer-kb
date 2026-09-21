# Optimization Budgets — Numbers That Bind

> Source of truth for triangle, material, texture, and memory budgets.
> Budgets are contractual: a task is not 'done' if the asset exceeds its tier.

## Triangle budgets (runtime mesh per LOD tier)

| Tier | Example | LOD0 | LOD1 (50%) | LOD2 (25%) | LOD3 (10%) |
|---|---|---|---|---|---|
| Hero character | playable/merc, protagonist | 30–60k | 15–30k | 7–15k | 3–6k |
| Hero prop | signature weapon, key item | 8–15k | 4–8k | 2–4k | <1.5k |
| Support prop | furniture, cover, doors | 2–5k | 1–2.5k | 0.5–1.2k | 0.2–0.5k |
| World block | modular cell kit pieces | 5–15k/block | 2.5–7k | 1.2–3.5k | impostor |
| Background/foliage | distant, scattered | 0.2–1k | — | billboard | impostor |

AI-mesh starting point note: raw output 50–500k for a prop → always converge to
tier above. Engine has no Nanite: explicit LODs required.

## Material / draw call budgets

- 1 material per prop · 1–3 per hero asset · 1 kit material set per world block.
- Per view budget (desktop-class): ≤ 2,000 draw calls; ≤ 1,500 on travel of the
  same world block. Instancing is the lever for scatter.

## Texture resolution by role (power-of-two)

| Role | Resolution | VRAM note |
|---|---|---|
| Hero character set | 2048 | 3 maps+normal ≈ 48 MB raw → ~6 MB KTX2 |
| Support prop | 1024 | |
| Background prop | 256–512 | |
| World block trim | 1024 | atlas per kit |
| Terrain megatextures | 4096 | tile-streamed |

## Memory targets

- Base GLB delivery: characters < 10 MB, props < 5 MB, world block < 20 MB
  (Draco + KTX2 optimized).
- Morph target discipline: sparse accessors; only active targets resident on GPU
  (see `core/03-animation.md` §4 memory math).
- Streamed world: resident cell budget ≈ 256 MB VRAM class at desktop;

## Streaming budget defaults (from world-creation)

- Terrain cells 1000 m · detail cells 500 m · props 250 m (loading radii).
- Unloading radius = loading radius + hysteresis band (flicker-free).
- Priority: ground → detail → props; preload spawn cells.

## Verification ritual (per asset, per task)

1. Triangle audit (viewport stats or bpy report) vs tier table.
2. Texture dimensions + count vs role table.
3. Draw call count (material count × instances) vs budget.
4. Optimized GLB size vs delivery targets.
5. One frame in-engine at worst-case viewpoint: report fps + VRAM.