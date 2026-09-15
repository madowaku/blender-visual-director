# Visual Evidence Bundle — v0.1 contract

A bundle ties the structured scene state, selected visual observations and review issues to the same production revision. It is a contract for minimal evidence, **not an implemented observer or MCP API**. v0.1 may assemble it from returned JSON, existing image content and manual review. Do not fabricate files or capabilities.

## Files selected by need

```text
evidence/<checkpoint-id>/
  scene.json
  diff.json
  front.webp
  side.webp
  three_quarter.webp
  camera.webp
  silhouette.webp
  issues.json
```

This is a menu, not a mandatory set. A camera/material checkpoint might contain only `scene.json`, `camera.png` and `issues.json`; a structural check may have only state. Keep the native supported image format (the audited MCP screenshot is PNG); `.webp` is an illustrative future option, not a conversion requirement. Avoid empty placeholder images or invented paths. Tool-returned images can instead be referenced by a durable ID if the host provides one. Without persistence support, keep an in-conversation evidence record and say it is not a persisted bundle.

## Minimum useful records

**`scene.json`** is the bundle manifest as well as scoped state:

- `schema_version` (`"0.1"`), `checkpoint_id`, `scene_id`, `revision`, UTC `captured_at`, Blender version and capability provenance.
- `scope`: objects/collections considered, total and returned counts, explicit `truncated`, coordinate/unit conventions, and `unobserved` fields. Include task-relevant transforms/bounds/materials/camera/render settings rather than entire meshes or node trees.
- `evidence`: for each **actually captured** image, record relative path or durable content ID, source (`mcp_viewport`, `blender_render`, `computer_use`, or an available observer), captured revision/time, view/camera identity, projection, resolution, shading/render engine and frame. Include color management when material/lighting judgment depends on it. Mark unavailable metadata `null` with a reason; do not assert inferred view settings as measured.
- `omitted`: requested but unavailable evidence and reasons. Views irrelevant to the checkpoint do not need entries.

`revision` can be a client-maintained batch number in v0.1, not a promised native Blender revision. Increment/invalidate it after relevant edits, including human edits if detected. If external changes cannot be tracked, re-query before final capture and record the uncertainty; timestamps alone cannot prove synchronized state.

**`diff.json`**, only with a comparable baseline: `baseline_checkpoint_id`, `checkpoint_id`, stable object identities (or scoped names with their limitations), `added`, `removed`, changed fields with before/after, and `unknown_changes`. Record which fields were compared. A camera-only change may invalidate every camera image without changing object geometry. Do not interpret a missing entry in a truncated query as deletion. If no baseline exists, omit the file and record `baseline unavailable`; do not invent an empty diff.

**`issues.json`**: checkpoint ID, review status (`unreviewed`, `partial`, `reviewed`), criteria actually reviewed, unobserved criteria, and an issue list. Each issue contains ID, object/region, criterion, impact, observed symptom, evidence reference, confidence, proposed MCP correction, verification view and resolution status. Link a resolved issue to the newer verification evidence. An empty list without `reviewed` status is not a clean bill of health.

## Capture discipline

Choose views from [visual-review.md](visual-review.md). Refresh affected state, capture selected images after the batch, and inspect them before recording conclusions. Preserve camera, selection, mode, shading, visibility and render settings when generating temporary views/passes; restore changes in `finally` and verify restoration. Use a temporary view/scene only when that is simpler and safe for the authorized project. Do not overwrite delivery output with preview renders.

Client-side JSON persistence avoids safe-mode file I/O restrictions inside Blender. If image generation or transport fails, retain the state and issue records with the missing-evidence reason. Failure is not an instruction to generate all other angles.

## Smallest useful future Visual Observer

1. Return a compact scoped state manifest with completeness flags and a caller-supplied revision/checkpoint ID.
2. Capture **only requested views** and return image bytes/durable references plus view, revision, shading, frame, color-management and capture-method metadata. Support camera render first; add front/side/three-quarter on demand.
3. Preserve temporary state and report restoration/capture failures. Return explicit stale, missing and unsupported status.

These are desired capabilities, not proposed callable names. Start with reliable image provenance and delivery; add stable identities, field diffs and change invalidation next. Automated aesthetic scoring, geometry heatmaps and always-on multi-view capture are outside the minimum.
