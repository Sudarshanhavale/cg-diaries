---
layout: page
title: "opec Vs Transparency"
permalink: /docs/opec-vs-transparency/
---

# Opec Vs Transparency

Opacity maps can slow down renders in ray tracing, especially at high resolutions. Use them only when they make sense for your asset and camera distance. The notes below will help you plan foliage and asset setup more efficiently.

## Quick Decision Tree
* **Close / Hero foliage:** Use real geometry (opaque). Only add transparency if absolutely needed.
* **Mid-distance foliage:** Use a mix: geometry for main shapes + transparency for filler. Disable or simplify shadows on transparent parts.
* **Far / Background foliage:** Use transparent planes or impostors. Turn off shadows or replace them with fake/baked ones.

---

## Guidelines: When to Use Which
* **Hero / Close / Mid Assets:**
  * Prefer opaque geometry (modeled leaves). This avoids heavy transparency shadow calculations.
  * If needed, mix with a cutout transparency on the less visible parts.
  * Possibly disable shadow casting on transparent objects and/or use proxy shadow meshes to reduce render cost.
* **Long-Distance / Background / Dense Areas:**
  * You can use transparent/cutout planes or impostors. But see if you can turn off real shadows and use fake shadows (blob, projected AO, baked).
  * Keep shaders simple: no transmission, no SSS.
  * For a fake SSS look on mid/far foliage, you can use Arnold’s double-sided shader.

---

## Best Practices & Tips
* Cutout (mask) transparency is cheaper than blended transparency.
* For large BG trees, use billboards or impostors.
* If transparency is required, disable shadows and use a simpler shadow proxy.
* Always test with a moving camera; distant tricks may break when seen up close.

---

## Short Comparison
| Factor      | Opaque Geometry                    | Transparent / Cutout Geometry                      |
| ----------- | ---------------------------------- | -------------------------------------------------- |
| Memory Cost | Higher (more polygons stored).     | Lower (fewer polygons).                            |
| Shader Cost | Cheaper, simpler shading.          | More complex shading.                              |
| Shadow Cost | Simple, efficient shadows.         | Expensive: transparency slows shadow rays.         |
| Render Cost | Faster: early depth optimizations. | Slower: overdraw + blending increases render time. |

---

## Rule of Thumb:
* **Opaque** = more memory, faster render.
* **Transparent** = less memory, slower render.