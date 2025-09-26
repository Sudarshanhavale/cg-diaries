---
layout: page
title: "Render Setting Tips"
permalink: /docs/render-setting-tips/
---

# Render Setting Tips

Always check render **log/stats.html** after  your renders.

## [Sampling](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_render_settings_ac_samples_html)
* Don’t just crank up Camera (AA) blindly.
* First, render with AA = 2 and check AOVs, find which ray type is noisy (diffuse, specular, transmission, SSS).
* Increase only that sample.
  * Example: Grainy reflections on metal → increase Specular samples, not Camera AA.

## Denoising
Always test denoising before raising samples.
* OptiX (GPU): Very fast, good for lookdev.
* OIDN (Intel CPU): High quality, reliable for finals.
* Noice (Arnold): Best for batch/sequence renders, supports temporal stability (no flicker).
Workflow: Render with low AA (2–3) >> run denoiser >> compare to high AA (6).

## [Light Samples](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_render_settings_ac_lights_settings_html)
* Keep non-key lights = 1.
* Key lights = 2–3, only if needed.
* Inspect Light Group AOVs: If noisy, increase only that light’s samples.


## [Adaptive Sampling](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_render_settings_ac_adaptive_sampling_html)
* Always enable on heavy frames to avoid unnecessary sampling on the part of the image that does not really need it. This could be one of the high render time-saving options for high-resolution frames.
* Saves time by giving more samples to noisy pixels and fewer to clean/flat pixels.
* Example: A frame with a bright, flat wall + a shiny metal ball.

| Without Adaptive Samples                    | With Adaptive Samples         |
| ------------------------------------------- | ----------------------------- |
| AA Samples = 6 (fixed)                      | Max AA = 6, Threshold = 0.015 |
| Every pixel = 36 samples, even a flat wall. | Flat wall ≈ 4 samples         |
| Shiny ball = 36 samples                     | Shiny ball ≈ 30+ samples      |
| Good quality, but slower render             | Same quality, faster render   |


## [Adaptive Subdivision](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_render_settings_ac_subdivision_html)
It's recommended to use adaptive subdivision all the time if you are going with more than 3 camera AA samples in render settings, so it prevents subdivision on unwanted areas, and controls the memory usage and render time.
* Object-level subdivision = Adds the same detail everywhere (wasteful).
* Adaptive subdivision = detail only where needed:
  * More splits → close to camera, curved surfaces, or displaced areas.
  * Fewer splits → flat or distant objects.

**Goal**: Keep detail where it matters: save render time and memory.

### Render Settings (Subdivision Section)
* **Max Subdivisions:** Caps the maximum level.
  * **Example:** If set to 2, no object subdivides more than 2 levels, even if a higher value is set at its object level by asset teams. Prevents huge memory use.
* **Frustum Culling + Padding:** Skips subdivision for objects outside the camera view.
  * Padding adds a small margin so objects don’t “pop” when they enter the frame.
* **Dicing Camera:** Provide render camera, OR use a custom dicing camera if you end up getting subdivision popping in the frame when using the render camera. 

`Always check render **log/stats.html**. If subdivision is taking too much time, lower it here.` 

## Low Light Threshold
`Adding details...`



---

## [Per-Object Adaptive Subdivision/Adaptive Metric](https://help.autodesk.com/view/ARNOL/ENU/?guid=arnold_user_guide_ac_polygons_ac_subdivision_settings_html)
You’ll find these controls in the Arnold section of the shape node’s Attribute Editor.
* Global settings (in Render Settings) affect all objects.
* These object-level parameters control subdivision only for that specific object.
* It’s recommended that the asset team sets these values when adding heavy subdivision to an object, so render time and memory are kept under control.

### Key Parameters:
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

### **How to validate adaptive subdivision value?**
In IPR, check the object at close, mid, and long distances using the Wireframe option in the RenderView. And adjust the adaptive values until the object keeps the necessary detail at all three camera ranges without wasting extra subdivisions.
1. In RenderView >> Render >> Debug Shading >> Wireframe (Turn ON).
2. Move the camera closer/farther.
3. Ensure mesh subdivides enough for close shots, but not wasteful in mid/long.

---

## HDRI Maps
* Lighting quality generated from 1k HDRI and 8k HDRI is identical in most cases. So use 1k whenever possible, it saves memory + I/O time.
* Use higher-res HDRIs only if visible in reflections.

---

## Golden Rules
* Samples: Raise selectively, not blindly.
* Denoise: Always test before increasing AA.
* Lights: Keep samples low, check Light AOVs.
* Adaptive Sampling: Faster renders, same quality.
* Adaptive Subdivision: Save memory + time, add detail only where needed.
* HDRIs: Use 1k for lighting, high-res only if seen in reflections.

---

##  Extra Tips from Forums & Docs
* Lazy vs full loading of stand-ins helps memory.
* The BVH build is one of the heavier steps; if your scene has lots of geometry, this is where optimizations matter.
* Adaptive subdivision (per-object + global) is crucial to avoid “exploding” polygon counts (especially with displacement).
* Procedural shading and transparency are heavy costs; many users on forums suggest baking heavy node networks when possible.
* Using one efficient shader across many objects, with variations driven via UVs or instance attributes, helps reduce shader overhead.
* Good bounding boxes are important for frustum culling, ensuring that off-camera or hidden objects aren’t needlessly processed.
* Always keep an eye on stats.html or render logs to see which steps (subdivision, shading, rays, textures) are taking the bulk of your time/memory.
---
