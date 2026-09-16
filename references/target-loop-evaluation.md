# Target-loop integration evaluation v0.2

Use this protocol to test whether target-image refinement plus Blender Visual Director improves visual progress per unit of observation/edit cost. This is an evaluation recipe, not runtime guidance to load on every Blender task.

## Fixed inputs

Freeze these before comparing conditions:

- one target image and stable `target_id`;
- one starting `.blend` scene;
- one task brief;
- Blender, BlenderMCP and model versions/settings;
- render engine, resolution, frame and color management;
- asset/network permissions;
- maximum refinement rounds or another shared stopping rule;
- one critic rubric and critic-model configuration.

Do not regenerate the target between arms. If the target changes, start a new comparison.

## Conditions

Run with separate agent contexts where practical.

### T0: target loop + ordinary BlenderMCP

Use the target and critic loop with normal BlenderMCP production behavior, without Blender Visual Director guidance.

### T1: target loop + Blender Visual Director

Use the same target/critic setup plus this skill. Computer Use is unavailable unless the ordinary non-CU image path fails so badly that the arm cannot complete; record any such deviation.

### T2: target loop + Blender Visual Director + extra sensor

Same as T1, but permit exactly the observation path being evaluated: a Visual Observer implementation if available, otherwise a justified Computer Use checkpoint. Do not add gratuitous sensor calls merely to exercise the condition.

## Challenge cases

A normal target-matching scene may never need an extra sensor. Include at least one controlled case that tests the observation boundary, such as:

- intentionally stale current-state image after a camera change;
- black/corrupted MCP capture;
- camera crop defect visible only in the final delivery framing;
- lighting defect that structured state cannot establish visually;
- scene-summary truncation above ten objects while a relevant object lies outside the returned summary;
- actual Blender modal/UI state when that state is itself the thing being verified.

Label injected defects. Do not silently alter one arm more than another.

## Round log

For every round, record:

- round and checkpoint ID;
- target ID;
- critic gaps entering the round;
- MCP calls, including failures/retries;
- planned edit batches and correction batches;
- scoped-state queries;
- generated/captured images;
- images actually inspected;
- visual review/critic events;
- evidence freshness (`fresh`, `cached`, `stale`, `uncertain`, `missing`, `unsupported`);
- Computer Use observations and GUI inputs;
- defect detection channel;
- defect resolution channel;
- final unresolved gaps;
- actual token usage if exposed, otherwise argument/response characters plus image count/resolution proxies.

Separate initialization/discovery from production and separate cold Skill/reference loading from warm refinement rounds.

## Visual comparison

Use the same independent critic configuration for all arms. Preserve each final render and each critic verdict. If the critic emits a score, treat it as one measurement rather than ground truth.

For stronger claims, add blinded final review using the same rubric and hide the arm identity. Compare at least composition, lighting, materials, details, silhouette/proportion where relevant, and obvious artifacts.

Pixel identity is meaningful only when the intended outputs should actually be identical. For target matching, the more useful question is whether high-impact visible gaps close faster or with less observation/edit overhead.

## Primary derived measures

Prefer descriptive metrics over a single composite score:

- high-impact critic gaps resolved per round;
- MCP calls per resolved gap;
- inspected images per resolved gap;
- Computer Use observations per resolved gap;
- number of repeated gaps across rounds;
- stale/invalid evidence incidents caught before judgment;
- final unresolved high-impact gaps;
- cold vs warm context cost.

Do not claim token savings when only character/image proxies are available.

## Observer success test

A minimum Visual Observer implementation passes its first practical milestone when, in T2:

1. a defect that would otherwise require a Computer Use observation is detected or verified using observer evidence;
2. the observer evidence is fresh and provenance-linked according to `visual-observer-spec.md`;
3. the same defect remains detectable at comparable confidence/quality;
4. the observer does not introduce more redundant image captures than it removes.

If T2 adds cost without changing a decision, record that honestly. The correct outcome may be that the existing Blender render path is already sufficient.

## Exit and next decision

After the trial, choose the next engineering step from evidence:

- If camera-render provenance/freshness solves most misses, implement only that.
- If scoped scene completeness is the blocker, improve scoped state before adding more views.
- If localized comparisons dominate cost, add safe crops from fresh captures.
- If UI state is the recurring blocker, retain Computer Use for that niche instead of reproducing the whole UI in an observer.
- If target-loop overhead dominates tiny tasks, add a threshold for when target matching should be activated rather than weakening the production loop.
