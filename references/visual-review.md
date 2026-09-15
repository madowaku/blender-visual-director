# Selective visual review

## Match the checkpoint to the change

Start with the question and one useful view. Add a second only when depth, overlap or ambiguity prevents judgment. Establish what “front” means for the asset instead of assuming a universal Blender axis.

| Change / checkpoint | Smallest useful evidence | Main questions |
| --- | --- | --- |
| Blockout complete | Three-quarter solid view | Overall form, proportion, balance, distinct parts |
| Major proportion change | Front **or** side matching the changed axis | Relative sizes, spacing and support; add the other axis only if unclear |
| Silhouette change | Relevant plain-background angle; silhouette pass only if ordinary shading hides the contour | Recognizable outline, tangencies, negative space |
| Materials / lighting | Intended camera with suitable material/render shading, usually a small render | Separation, exposure, shadow readability, surface artifacts |
| Camera composition | Current camera view; render if crop/effects matter | Framing, margins, subject hierarchy, clipping, overlaps |
| Final review | Fresh deliverable view/render | User brief and remaining issues; add hidden-side views only if the deliverable requires them |

Material and lighting changes in one batch can share one image. A name-only edit often needs no new image; preserve the link between prior evidence and renamed objects. A transform, camera, shader, frame or render-setting change invalidates affected visual evidence.

## Review language

Assess relevant criteria; mark the rest unobserved or not applicable.

- **Silhouette:** contour recognition and clean negative spaces at intended display size.
- **Proportion:** relationships between parts, thickness and length, relative to the brief.
- **Shape language:** consistent curves, angularity, bevels, repetition and detail density.
- **Balance:** apparent support, visual weight and directional pull.
- **Readability:** main subject and secondary features remain distinguishable; no accidental mergers.
- **Camera composition:** crop, margins, hierarchy, focal emphasis and distracting tangencies.
- **Material separation:** intended surfaces remain distinct under delivery lighting. Different material names alone are insufficient.
- **Lighting:** form reads, focal areas have usable exposure, and shadows support depth.
- **Obvious geometry artifacts:** visible holes, intersections, faceting, z-fighting, floating parts or modifier failures. Infer hidden topology only from appropriate structured checks.

Record a defect as **location + visible symptom + effect on intent + likely fix + verification view**. Example: “Camera view: right arm merges with torso, hiding its pose; move the arm outward through MCP and recheck this camera.” Separate an observed symptom from a suspected cause; record uncertainty when a shadow could be mistaken for a hole.

Fix high-impact defects together, then recheck only affected views. Stop when the requested quality and deliverable are met; avoid unrequested polish loops. If a repeated fix does not improve the same symptom, change the diagnosis or request missing intent rather than repeating it.

## Computer Use as sensor

Use it when the actual UI must be seen or the cheaper image routes remain inadequate. Capture only the target window/area if supported, at a resolution that answers the question. Prefer one observation that answers several related questions; never lower resolution until the defect becomes invisible.

Inspect the returned image, its actual view and shading mode. A success flag, filepath or base64 string is not visual inspection. A solid viewport does not validate final materials or lighting. A saved render can be stale even when its filename is correct.

If GUI input is required to reveal a viewport, dismiss an understood non-sensitive obstruction, or open Render Result, use the active Computer Use tool's fresh-state rules for each input. The saving comes from fewer GUI actions, not stale coordinates or skipping required refreshes. Do production fixes through MCP whenever possible.
