---
layout: page
title: "Shader Optimization Tips"
permalink: /docs/shader-optimization-tips/
---

# Shader Optimization Tips

## Big Picture
* **Procedural nodes** (aiCellNoise, aiTriplanar, aiColorJitter, layered): These are flexible but slow, especially with transparency.
* **Baked textures:** faster, lighter, predictable, ideal for large environments and high-res renders.

```
Use procedural only when you need flexibility. Otherwise, bake!
```
---

## Guidelines by Distance
#### **Close / Hero Assets**
  * Use procedural nodes only if you need controls (e.g. color variation, breakup, shot base changes).
  * Always consider baking stable patterns (bark, walls, roofs), you can keep work file and publish file separate if needed.
  * Randomwalk SSS, only if important, else swap to Diffusion SSS or fake with textures/backligh/double-sided shaders.
  * Bake layered shaders (diffuse, roughness, specular), plug into clean standard_surface.
  * Maps: 1k–2k (use higher only if justified).

#### **Mid-range Assets**
  * Bake procedural shading nodes (noise, triplanar, layered materials).
  * Avoid procedural opacity maps and use model cutouts instead.
  * Replace Randomwalk SSS with transmission/double-sided shader if that works for you.
  * Combine geometry, fewer objects, and fewer shading calls.eg: a tree asset can have a single material and leaves, branches and trunk can be in the same uv space or use a UDIM.  OR you can go with Arnold filename tokens in some cases.
  * Maps: 1k or 2k maps are usually enough, but you can review renders with the comp team to decide whether you want to go with high resolutions in some areas.

#### **Far / Background Assets**
  * Possibly, you can use fully baked textures and keep shaders very simple (no heavy nodes).
  * Transparency is ok if its needed, but disable shadows (use fake/baked shadows).
  * Maps: Check if 0.5k–1k resolution holds for your camera.

---

## SSS (Subsurface Scattering) Cheat Sheet
* **Randomwalk SSS:** Hero assets, closeups, action shots.
* **Diffusion SSS:** Mid-range, fast-moving elements, cheaper but good enough.
* **Transmission / Double-sided shader:** Background, cheap alternatives if that holds in your frame.

---

## Bake TX
* Bake tx before sending data to lighting. Instead of relaying on pre render tx baking.
* Also tx map size are usually low in comparison with original textures, so I/O loading will be low.

---

## Extra Optimization Tricks
* Normal maps Vs displacement (use displacement only if absolutely required). Eg: City houses wal can have normal pam instead of displacements.
* In my experience, we have baked all shaders before the final 4k renders and that helped us saving lots of render cost.
* Reduce shader count:
  * One shader can drive leaves + branches + trunk (shared UVs/UDIMs).
* [Filename tokens](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_textures_ac_filename_tokens_html): Allows one shader for many objects (be mindful of per-shape map resolution overhead). Read more into [here](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_textures_ac_filename_tokens_html).

---

## Best Practices
* Bake procedural shader once, reuse many times.
* Layered materials: bake the final look into maps, unless the shot overrides are needed.
* Opacity maps: avoid for close/mid foliage; use only in BG (shadows off if possible).
* Fake translucency: baked diffuse/transmission maps or backlight AOVs.
* Combine shapes: background trees shouldn’t be split into many small parts.

---

## Rule of Thumb:
* **Procedural** = flexible, but expensive at scale.
* **Baked** = less flexible, but far cheaper for rendering.

---
