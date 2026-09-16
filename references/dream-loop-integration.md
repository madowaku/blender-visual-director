# Dream Loop interoperability

Use this reference when a user supplies a target image, asks for a Dream Loop-style visual refinement loop, or another skill/orchestrator is already generating a target and judging the current result.

This skill remains the Blender production director. It does **not** absorb the target-generation or critic workflow. The goal is a clean handoff:

```text
target / critic
      ↓
blender-visual-director
      ↓
   BlenderMCP
      ↓
    Blender
      ↓
minimal fresh evidence
      └────────→ critic
```

The integration is compatible with the ideas in [achimala/dream-loop](https://github.com/achimala/dream-loop) without vendoring or requiring that skill. If Dream Loop is installed, let it own its own orchestration policy and exit criteria.

## Responsibility boundary

- **Target loop / critic:** owns the target image, target version, comparison rubric, round count and stop/rethink decision.
- **Blender Visual Director:** translates visual gaps into coherent Blender edits, chooses the smallest useful verification evidence, and protects evidence freshness.
- **BlenderMCP / Blender Python:** performs production edits and structured state queries.
- **Visual Observer or existing image path:** captures evidence with provenance.
- **Computer Use:** fallback sensor for UI/modal state or when cheaper image paths cannot answer the visual question.

Do not run a second independent aesthetic judge inside this skill merely because a target-loop judge exists. Avoid duplicated screenshots through multiple channels.

## Round contract

For each refinement round:

1. Receive the immutable `target_ref` for this round and the latest critic gaps, if any.
2. Convert the gaps into concrete observations: what is visibly wrong, which Blender state is likely relevant, and which view can verify a fix.
3. Group related corrections into the smallest coherent MCP edit batch. Prefer high-impact composition/proportion/lighting fixes before tiny detail edits.
4. Query scoped state after the edit. A successful mutation is not visual proof.
5. Capture only the fresh evidence needed to answer the verification question.
6. Return the evidence and a compact round summary to the outer critic. Let that critic decide whether another round is needed.

When no critic feedback exists yet, treat the target as visual intent and use the intended delivery camera/render as the default first comparison view.

## Critic-gap handoff

Prefer gaps shaped like this, whether represented as JSON or concise text:

```json
{
  "gap_id": "lighting-face-01",
  "criterion": "lighting",
  "target_region": "face",
  "symptom": "face is flatter and darker than target",
  "requested_outcome": "restore brighter key-side modeling without clipping",
  "priority": "high",
  "confidence": 0.9,
  "evidence_ref": "round-2-camera"
}
```

Treat `symptom` as observed evidence and the proposed cause/fix as a hypothesis. Re-diagnose when a repeated correction does not improve the same symptom.

Map common target-comparison criteria to evidence deliberately:

| Critic criterion | Default evidence | Add more only when |
| --- | --- | --- |
| Composition | Intended camera render/view | depth/occlusion makes placement ambiguous |
| Lighting | Intended camera render with delivery engine/color management | a local region needs a crop or alternate diagnostic view |
| Materials | Intended camera render | target surface is hidden or the render cannot distinguish material from lighting |
| Detail | Relevant delivery view or crop | geometry vs texture ambiguity requires structured state/alternate view |
| Silhouette / proportion | Relevant target-facing view | target perspective hides the changed axis |

A crop may reduce visual payload for a localized question, but retain enough surrounding context to judge the relationship being fixed.

## Target and evidence freshness

Give each target a stable `target_id` or content hash when possible. A regenerated target is a new target version and invalidates prior critic comparisons, even if the scene did not change.

Every current-state image used for comparison should be linked to the evidence fields in [evidence-bundle.md](evidence-bundle.md): checkpoint/revision, camera/view identity, render settings, frame, resolution, color management when relevant, and capture method. Do not compare a target against an image known to predate a relevant edit.

A useful cache key is conceptually:

```text
scene revision
+ view/camera signature
+ frame
+ visibility state
+ render/shading settings
+ color management
```

Reuse evidence only when all fields relevant to the comparison are unchanged. If external scene changes cannot be ruled out, mark freshness uncertain and re-query or recapture before a final judgment.

## Cost discipline

Target matching can become an expensive screenshot metronome. Keep the loop sparse:

- one coherent edit batch may address several related critic gaps;
- one delivery render may validate composition, lighting and material separation together;
- do not recapture an unaffected orthographic view after a camera-only change;
- do not use Computer Use simply because a target comparison exists;
- separate cold skill/reference loading cost from warm iteration cost in evaluations.

Optimization objective: **maximize meaningful visual-gap reduction per observation/edit round**, not minimize Computer Use at any cost and not maximize critic score through redundant captures.

## Stall handoff

If the same high-impact visible gap survives two materially different correction attempts, stop micro-tuning that symptom. Return a stall note to the outer loop with:

- the persistent symptom;
- evidence across the failed attempts;
- hypotheses already tried;
- the suspected architectural decision that may need rethinking (camera, asset, topology, lighting rig, material model, etc.).

Let the outer target loop decide whether to make a larger redesign, regenerate the target, or stop.

## Evaluation

For a meaningful integration test, keep the same target, starting `.blend`, Blender/MCP versions, render settings, stopping rule and critic rubric across conditions. Compare at least:

- target loop + ordinary BlenderMCP policy;
- target loop + Blender Visual Director;
- target loop + Blender Visual Director + one justified extra sensor path (Visual Observer or Computer Use).

Use independent contexts and a fresh critic where possible. Record rounds, MCP calls, edit/correction batches, generated and inspected images, review events, Computer Use operations, failed calls, actual token usage if exposed, and cold/warm context cost. A critic score is supporting evidence, not a substitute for preserving the actual final images and defect history.
