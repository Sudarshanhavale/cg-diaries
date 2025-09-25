---
layout: page
title: "Asset Planning and Instancing"
permalink: /docs/asset-planning-instancing/
---

# Asset Planning and Instancing

One of the most important point is right asset planning and instancing of the repetitive components 
so we don't end up utilizing extra resources than we suppose to use originally.

## Instancing
### What is Instancing?
Instancing means **reusing the same geometry file** many times in a scene without duplicating it in memory.
* Saves memory: One copy stored, many placements reused.
* Faster renders: Arnold doesn’t reload identical data.
* Best for environments, foliage, crowds, and repeating props.

---

### Types of Instancing

#### Procedural Instancing (MASH, XGen, Bifrost)
* Creates huge numbers of copies.
* Perfect for **heavy population:** forests, grass, debris, crowds.
* Combine with **frustum culling + LODs** to stay efficient.

#### Arnold Auto-Instancing (Standins / USDs / GPU nodes)
* If multiple standins use the same .ass or .usd, Arnold loads it once and reuses it everywhere. (There is an option in the stand-in shape to turn this OFF)
* This is the most optimized, with no extra steps for artists.
* Best for hero props, action items, camera-framed elements, where placement must be artistic but still memory-friendly.

#### Maya Reference Instancing
* Maya’s native referencing is good for large set builds.
* The modeling team can reference props/components,  Arnold auto-instances at render.

---

### Who Uses It?
* **Modeling team:** place props/standins/GPUs, Arnold auto-instances at render time.
* **Artists:** use procedural instancing for mass layouts; use standins or references for hero/camera objects.
* **Previewing:** standins can be shown as a bounding box, shaded, or wireframe in Maya, so the modeling team doesn’t have to worry too much about visual reference while working.

## Subdivision Tip:
```
For instanced assets, set Adaptive Subdivision → Object Space, so detail doesn’t “pop” when the camera moves.
```

## Best Practices
### Geometry
* Keep minimum object count, combine shapes (e.g., one mesh per tree instead of many leaf meshes).
* Avoid super-thin geometry for BG foliage (causes flickering).
* Use lores geometry or cutout geometry instead of opacity maps for mid/close foliage.
* Use a mid/low poly model and rely on subdivision so that LODs can be controlled in lighting.
* Use LODs wherever possible.

### Shading
* Use one simple shading network per instanced prop.
* Shader variations: drive with UVs/UDIMs or instance attributes (color, roughness) instead of extra shaders.
* Prefer normal maps over displacement (use displacement only if really needed).
* Bake procedural shading (noise, triplanar, jitter) into maps.
* Always bake instancing into standins (.ass/.usd).
* Use MaterialX when possible; lighters can override with aiOperators at render.

### Textures
* Use lower-res maps for BG (0.5k–1k), mid assets at 1k–2k.
* Avoid unnecessary 4k maps unless seen up close.

---

## Frustum Culling & LODs
* Procedural tools (MASH, XGen, Bifrost) support frustum culling, only render what’s visible in the camera. Some custom tools can make it artist-friendly.
* Use LODs:
  * Close = full detail.
  * Mid = simplified.
  * Far = very low poly or billboard/impostor.
* Ensure standins have correct bounding boxes so culling works correctly.

---

## Extra Tips
* Shader cost vs geometry: More geometry + simple shader is often faster than fewer polys + heavy shader (especially with transparency).
* Fast Point Instancing is lighter than full instancing (HtoA/Arnold).
* Slightly larger leaves in BG reduce flickering/aliasing.
* Transparent foliage is costly in shadow rays; use geometry for mid/close, transparency only for distant BG with shadows disabled.

---

## Golden Rules
* Procedural instancing = massive populations (grass, forests, debris).
* Auto/Reference instancing = hero, camera, artistic layouts.
* Duplicate = heavy. Instance = light.
* Transparency = slow. Geometry = heavier memory but faster render.
* Use frustum culling + LODs in all large environments.
* Fewer objects + fewer shaders = faster renders.

---

## Subdivision

### [Per-Object Adaptive Subdivision/Adaptive Metric](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_polygons_ac_subdivision_settings_html)
You’ll find these controls in the Arnold section of the shape node’s Attribute Editor.
* Global settings (in Render Settings) affect all objects.
* These object-level parameters control subdivision only for that specific object.
* It’s recommended that the asset team sets these values when adding heavy subdivision to an object, so render time and memory are kept under control.

#### Key Parameters:
* Adaptive Metric:
  * _flatness_ = adds detail in curved areas.
  * _edge_length_ = adds detail on edges close to the camera.
  * _auto_ = Arnold decides (best with displacement).
* Adaptive Error: Lower value keeps more subdivisions (heavier). Higher value keeps fewer subdivisions (lighter).
* Adaptive Space:
  * _raster_ = camera distance >> closer objects subdivide more.
  * _object_ = object units >> detail is fixed regardless of distance. (Use this for instance objects to avoid popping)
* UV Smoothing: How UVs behave when subdividing (pin_corners, linear, smooth).
* Ignore Frustum Culling: Useful if the object is off-screen but seen in reflections.

#### **How to validate adaptive subdivision value?**
In IPR, check the object at close, mid, and long distances using the Wireframe option in the RenderView. And adjust the adaptive values until the object keeps the necessary detail at all three camera ranges without wasting extra subdivisions.
1. In RenderView >> Render >> Debug Shading >> Wireframe (Turn ON).
2. Move the camera closer/farther.
3. Ensure mesh subdivides enough for close shots, but not wasteful in mid/long.

---


