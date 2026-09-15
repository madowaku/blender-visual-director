---
name: blender-visual-director
description: Direct Blender 3D creation and revision with BlenderMCP, batching edits and choosing minimal visual evidence for quality review. Use for modeling, materials, lighting, and camera work where visual quality matters.
metadata:
  short-description: BlenderMCP production with selective visual review
---

# Blender Visual Director

Use BlenderMCP as the main editing channel. This skill directs production and observation; it does not replace BlenderMCP or install an observer. Treat Computer Use primarily as a visual sensor.

## Establish capabilities once

Inspect the current callable tools and their input schemas; map editing, scene queries, and image retrieval to tools actually exposed. Confirm the connected Blender version and scene before editing. Recheck capabilities after a connection or version change, not every batch.

Read [blender-mcp-strategy.md](references/blender-mcp-strategy.md) for the audited API, query limitations, batching, or connection failures. Its names describe a verified upstream version, not guaranteed tools in this session. Do not invent endpoints or silently replace a missing MCP connection with GUI production.

## Production loop

1. **Plan:** Identify the deliverable, objects to change, visual intent, and the next useful checkpoint. Choose one coherent batch with a verifiable outcome.
2. **Edit through MCP:** Group related geometry, transforms, materials, or camera changes. Preserve unrelated scene content. Prefer bounded, rerunnable Blender Python over individual GUI operations.
3. **Check structured state:** Confirm the batch's intended outcome and errors with a scoped query. Return compact JSON from Python when the standard query omits needed data. Structural success alone does not establish visual quality.
4. **Observe when it resolves a visual question:** Select the smallest adequate view and modality. Reuse existing evidence only if the scene, view, and settings relevant to the question are unchanged.
5. **Correct:** Describe visible defects concretely, group their fixes through MCP, check the affected state, and reobserve the views invalidated by those fixes.
6. **Final visual review:** Inspect fresh evidence of the deliverable after the last relevant edit. For a final render, inspect that render. Report unobserved aspects explicitly if image retrieval is unavailable; do not claim visual approval from JSON alone.

## Choose observation deliberately

Checkpoint candidates: completed blockout; major proportion changes; silhouette changes; material or lighting changes; camera composition changes; final review. Combine overlapping checkpoints and select only the views needed for the change.

Prefer an adequate existing image, then a verified MCP image or readable Blender render. A future Visual Observer may supply the same evidence contract when available. Use Computer Use when those paths cannot answer the visual question: actual UI/modal state, a missing or defective image path, an uncertain viewport, or a requested check of the displayed Blender result.

Before capture, name the question the image will answer. Do not alternate every MCP edit with a screenshot or Computer Use call. Avoid redundant captures through two channels. If GUI input is necessary to expose evidence or recover the connection, keep it targeted, follow that tool's observation rules, and return to MCP editing. Do not skip required input refreshes to lower the counter.

Read [visual-review.md](references/visual-review.md) when selecting views, judging quality, or diagnosing an image. It covers silhouette, proportion, shape language, balance, readability, composition, material separation, lighting, and obvious geometry artifacts.

## Evidence and evaluation

For persisted review evidence or a future observer integration, read [evidence-bundle.md](references/evidence-bundle.md). A Visual Evidence Bundle is a minimal, revision-linked set of state and selected images, not an instruction to generate every possible view. v0.1 can use tool-returned images and manually recorded issues.

For A/B/C comparisons or measured efficiency claims, read [evaluation.md](references/evaluation.md). Count MCP calls, Computer Use operations, image captures, actual visual reviews, and correction batches separately. Never equate zero Computer Use with good quality or invent token measurements.
