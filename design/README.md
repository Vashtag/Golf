# Handoff: Dusk Drive — Asset Library & Visual System

## Overview
Dusk Drive is a mobile arcade golf/flinging game in the spirit of Alto's Odyssey — side-scrolling, silhouette foreground against lush atmospheric skies. This handoff covers the **asset library / visual system catalog** used by art and engineering to agree on what the world looks like: trees, ground surfaces, skies (biome × time of day), and background structures.

## About the Design Files
The files in this bundle are **design references created in HTML/Canvas 2D** — working prototypes that show the intended look and motion behavior, not production code to ship directly. The task is to **recreate these designs in the target game engine** (Unity, Godot, Cocos, SpriteKit, WebGL, etc.) using its established rendering patterns, sprite pipelines, and scene composition tools.

If no engine has been chosen yet, these mocks are framework-agnostic enough to port to any 2D engine that supports per-layer parallax, alpha-blended silhouettes, and particle systems.

## Fidelity
**High-fidelity visual targets, low-fidelity implementation.**
- Colors, silhouette shapes, rim-light placement, dust/haze behavior, and biome palettes are authoritative — match them exactly.
- The Canvas 2D drawing code is a *reference implementation*, not a spec. In a real engine you'd typically use hand-painted PNG sprites, Spine/Skeletal rigs for windmill/tree sway, and shader-based sky gradients — not procedural canvas calls.

## Catalog Sections

### 01 · Trees (silhouette foreground)
6 species, each drawn as pure-black silhouette with subtle **rim light** (biome-dependent color) on the sun-facing edge.

| Name     | Biome(s)         | Notes |
|----------|------------------|-------|
| Palm     | Dunes · Oasis    | Curved tapered trunk, 7 fronds, each frond has 9 leaflet pairs feathering out |
| Saguaro  | Dunes · Mesas    | 3-part body (trunk + left arm + right arm), vertical ribs, spine detail |
| Deadwood | Crags            | Gnarled main trunk + 8 quadratic branches + random twigs |
| Pine     | Hills            | 6 tiers of serrated triangular foliage, internal needle strokes |
| Snowpine | Hills · Snow     | Pine with zigzag white snow cap on each tier |
| Juniper  | Mesas            | 9 irregular scalloped blob clusters |

**Wind sway:** global wind parameter (0–2×) drives a sinusoidal horizontal offset applied to canopy/fronds; trunk stays rooted. Each species uses its char-code as phase offset so they sway out of sync.

### 02 · Ground surfaces
6 tiling textures. Each is a vertical gradient + species-specific detail pass. Stats shown (friction/restitution) are placeholder physics values — tune in engine.

| Name    | Character |
|---------|-----------|
| Fairway | Deep green with dappled light patches + 80 animated swaying blades + tiny flowers |
| Rough   | Darker, tangled crossed blades + scattered twigs |
| Sand    | Warm dune gradient + 14 animated ripple lines + grain speckles |
| Green   | Striped mowing pattern + short blades + wet sheen highlight + dew |
| Rock    | Weathered stone with embedded pebbles + cracks + gravel specks |
| Ice     | Glacial blue with animated sheen sweep + fracture network + trapped bubbles + frost crystals |

### 03 · Skies (4 biomes × 3 times)
The hero shot. Full atmospheric parallax scene at 2.4:1 aspect ratio.

**Biomes:** Dunes (warm orange), Crags (pink/magenta), Hills (cool blue), Mesas (terracotta)
**Times:** Clear (sun high), Dusk (sun low, long bloom), Night (moon, stars, fireflies)

**Layer stack (back to front):**
1. Vertical gradient sky (3-stop, biome+time specific)
2. Stars (night: 120, dusk: 40; twinkle via sin(t))
3. Sun or moon with bloom/halo (moon has craters + halo gradient; sun has bloom gradient + core)
4. 4 drifting painterly clouds (soft radial gradients, biome-tinted underbellies)
5. 24 dust motes (hills: 10, crags: 12) drifting right with vertical sway
6. 6 parallax hill layers, each with: haze gradient behind + silhouette fill + rim-light stroke on top edge
7. Bottom haze gradient (matches biome haze color, 18% opacity)
8. Fireflies (night only): 14 glowing green dots pulsing with cubic sine

### 07 · Background structures
4 detailed silhouettes. All match tree rim-light convention.

| Name         | Detail |
|--------------|--------|
| Windmill     | Trapezoidal lattice tower with X-braces + tail fin + 14-blade wheel rotating at `millSpeed × 0.9 rad/s` |
| Grain Silo   | Cylinder with 5 weathering bands + dome cap + right-side ladder with 12 rungs + small window |
| Water Tower  | 4 tapered legs + 3 tiers of X-braces + conical tank + lightning rod finial |
| Radio Tower  | 8-segment tapered lattice + antenna spire + side dish + guy wires + red blinking beacon (`sin(t*3) > 0.2`) |

## Biome × Time Palette (authoritative)

All biomes share the same keys: `sky[3]`, `rim`, `haze`, `ground`.

```js
const BIOMES = {
  dunes: {
    clear: {sky:['#ffe6ba','#f38e41','#9d3e25'], rim:'#ffb878', haze:'#d4886a', ground:'#1a0d08'},
    dusk:  {sky:['#ffcbb6','#c85a4e','#4a1a2a'], rim:'#ff8866', haze:'#9a4858', ground:'#0d0508'},
    night: {sky:['#1e2a55','#0f1a38','#050812'], rim:'#4a6aa0', haze:'#2a3a60', ground:'#020408'},
  },
  crags: {
    clear: {sky:['#f8c3da','#d5425a','#5b1a2b'], rim:'#ff9ab0', haze:'#b07080', ground:'#180510'},
    dusk:  {sky:['#e8a0c0','#a02a48','#380812'], rim:'#d06080', haze:'#803050', ground:'#0c0208'},
    night: {sky:['#1a1828','#0a0818','#020008'], rim:'#4a3a58', haze:'#20182a', ground:'#020006'},
  },
  hills: {
    clear: {sky:['#cfe2ff','#5d79b0','#233d6d'], rim:'#a8c8ff', haze:'#8098c0', ground:'#0a1420'},
    dusk:  {sky:['#f0b8a8','#8878b0','#1a2050'], rim:'#c898d8', haze:'#6060a0', ground:'#050818'},
    night: {sky:['#0a1830','#050c1a','#01030a'], rim:'#3050a0', haze:'#081830', ground:'#000308'},
  },
  mesas: {
    clear: {sky:['#ffe1b0','#cf7b35','#6e3b1b'], rim:'#ffc070', haze:'#b87850', ground:'#1a1008'},
    dusk:  {sky:['#ffb888','#b04028','#380810'], rim:'#ff8050', haze:'#883830', ground:'#0d0408'},
    night: {sky:['#1a1a2a','#0a0a18','#020208'], rim:'#4a4060', haze:'#1a1828', ground:'#020006'},
  },
};
```

**Silhouette fill:** `#050607` (not pure black — has a faint blue tint that reads as "dark" at any brightness).

## Design Tokens

**UI chrome (for the catalog app itself):**
- bg `#07040a` · surface `#0f0814` · surface-2 `#170e1e`
- border `rgba(255,138,61,.1)` · accent `#ff8a3d` · text `#f0e8e0` · muted `#6a5e72`
- Fonts: DM Sans (400/500/600/700/800) · DM Mono (mono labels)
- Radius: 8px (controls), 12–16px (cards)

## Interactions & Behavior

**Render architecture** (matters regardless of engine):
- Single central RAF/game loop; each card/scene registers a `draw(t)` callback.
- **Skip any layer whose render target has zero size** (fixes hidden-tab / offscreen issues).
- All animation is time-based (`t = elapsed seconds`), not frame-based — resilient to variable framerate.

**Tweakable controls exposed in the catalog:**
- Trees panel: background biome, time-of-day, wind slider (0–2×)
- Skies panel: biome, time-of-day
- Structures panel: background biome, time-of-day, windmill speed slider (0.2–4×)

## State Management
- `biome: 'dunes' | 'crags' | 'hills' | 'mesas'`
- `time: 'clear' | 'dusk' | 'night'`
- `wind: number` (tree sway multiplier)
- `millSpeed: number` (windmill rotation multiplier)
- In the real game these would be driven by world position + in-game time, not by UI.

## Assets
All imagery is **procedurally drawn in Canvas 2D** in the prototype. For production, commission or paint PNG sprites:
- 6 tree silhouettes × 3 time-of-day rim-light variants = 18 PNGs
- 4 structure silhouettes × 3 rim-light variants = 12 PNGs
- 6 ground textures as tileable 512×512 PNGs (or 3-slice with animated blade layer on top)
- Cloud sprites × 4 shapes (alpha-blended)
- Dust/firefly particle sprites

## Files
- `Dusk Drive - Assets.html` — the interactive asset catalog (4 tabs, all renderers)
- `Dusk Drive - Design Catalog.html` — earlier higher-level catalog (included for context)

## Implementation Notes
- Don't port the procedural canvas code line-by-line. Use it to extract exact palettes, layer counts, motion timing, and parallax ratios (the 5–6 hill layers use par-factor `0.04 + i*0.05` and scroll speed `t * 3.5 * par`).
- Rim light is **always** on the top/sun-facing edge of silhouettes — lifting a 1px stroke in the biome's rim color at ~30% opacity is what sells the Alto look.
- Haze gradient behind non-first hill layers (18–30% of biome haze color) is what creates apparent atmospheric depth. Do not skip it.
- `#050607` silhouette fill, not `#000000`. Checked against Alto reference.
