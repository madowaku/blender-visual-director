# Blender Visual Observer — minimum contract v0.1

This document specifies the smallest observation layer that can reduce expensive GUI/Computer Use checks without weakening visual verification. It is a capability contract, not a commitment to a particular MCP tool name, server architecture, or Blender addon.

## Goal

Given a known production checkpoint, return **fresh, provenance-linked Blender state and only the requested visual evidence**. The observer does not decide whether the art is good. It makes trustworthy evidence cheap enough for another model/critic to judge.

The first implementation should optimize for one thing: replacing avoidable Computer Use observations in target matching and final visual checks.

## Non-goals

v0.1 should not:

- autonomously score aesthetics;
- continuously stream the viewport;
- capture every orthographic angle after every edit;
- duplicate BlenderMCP editing functions;
- hide missing metadata by guessing;
- become a second orchestration agent.

## Request contract

A logical observation request should contain the equivalent of:

```json
{
  "checkpoint_id": "round-3-after-lighting",
  "revision": 12,
  "purpose": "compare delivery lighting to target",
  "views": [
    {
      "kind": "camera",
      "camera": "Camera",
      "mode": "render"
    }
  ],
  "state_scope": ["Camera", "KeyLight", "FillLight", "Robot"],
  "target_ref": "optional-target-id",
  "allow_cached": true
}
```

`target_ref` is metadata only. The observer must not perform the visual judgment merely because a target exists.

The caller may omit `revision` if no reliable revision counter exists, but the response must then report freshness uncertainty and enough state metadata to support revalidation.

## Required v0.1 response

Return a compact manifest with:

```json
{
  "schema_version": "0.1",
  "checkpoint_id": "round-3-after-lighting",
  "requested_revision": 12,
  "observed_revision": 12,
  "freshness": "fresh",
  "scene": {
    "blender_version": "...",
    "frame": 1,
    "scoped_state": {},
    "truncated": false
  },
  "captures": [
    {
      "id": "camera-render-12",
      "kind": "camera",
      "source": "blender_render",
      "content_ref": "...",
      "camera": "Camera",
      "projection": "PERSP",
      "resolution": [640, 640],
      "render_engine": "BLENDER_EEVEE_NEXT",
      "frame": 1,
      "color_management": {},
      "captured_revision": 12,
      "freshness": "fresh"
    }
  ],
  "omitted": [],
  "warnings": []
}
```

The exact transport may return bytes, a host image object, a durable content ID, or a safe file reference. The manifest must make clear which one was returned. Never claim an image was visually inspected merely because it was captured successfully.

## Minimum capabilities

### 1. Scoped state

Return the task-relevant scene state without the standard scene-summary truncation becoming invisible. Include:

- total matching object count and returned count;
- explicit `truncated` / `complete` status;
- camera transform and lens/projection when camera evidence is requested;
- frame and relevant render/shading settings;
- task-relevant object transforms/bounds/material identities when requested;
- Blender version and capability provenance.

Do not dump full meshes or full node graphs unless the caller explicitly needs them.

### 2. Delivery camera capture

This is the highest-priority visual path for v0.1.

Capture the intended camera through the render/shading path appropriate to the verification question. Return image content plus camera, resolution, frame, engine, color-management and revision metadata.

Temporary preview settings must be restored even when capture fails. Never overwrite the user's final delivery output merely to create evidence.

### 3. Explicit freshness

Return one of:

- `fresh`: captured after the stated relevant revision with matching view/settings;
- `cached`: safely reused because all relevant cache-key fields match;
- `stale`: known to predate a relevant edit or setting change;
- `uncertain`: external changes or incomplete metadata prevent proof of freshness;
- `missing`: capture did not succeed;
- `unsupported`: requested view/mode is unavailable.

The caller should not need to infer freshness from timestamps alone.

## Cache and invalidation

A capture may be reused only when all fields relevant to its purpose are unchanged. Conceptually key cached evidence by:

```text
scene revision
+ camera/view signature
+ frame
+ visibility state
+ render or shading settings
+ resolution
+ color management
```

Examples:

- a camera transform change invalidates camera-composition evidence;
- a material change invalidates material/lighting evidence for affected visible objects;
- a camera-only change need not invalidate an unrelated front orthographic geometry check;
- renaming an object need not invalidate pixels, but the evidence manifest must preserve identity linkage;
- a target-image change does not make the current Blender capture stale, but it invalidates prior target-comparison verdicts.

When invalidation cannot be computed safely, prefer `uncertain` over silently reusing evidence.

## Optional v0.1 extensions

Add these only after camera capture + freshness are reliable:

1. front/side/three-quarter solid views on request;
2. plain-background silhouette capture;
3. region crop derived from an already-fresh capture;
4. stable object identities and scoped field diffs;
5. visibility-aware bounds/framing diagnostics.

Do not implement all of them merely because the schema can describe them.

## Computer Use fallback

The observer should make the fallback decision legible to the caller.

Escalate to Computer Use when:

- the actual Blender UI/modal state is the subject of the check;
- the observer capture path is missing, black, corrupted, or ambiguous after reasonable retry/recovery;
- the user explicitly asks to inspect what is displayed in the Blender window;
- a host/tool integration failure prevents otherwise required visual proof.

Do not escalate simply because a target image exists. If a Blender render answers the question, prefer it.

## Dream Loop / target-matching compatibility

When used with [dream-loop-integration.md](dream-loop-integration.md), the outer critic supplies the comparison goal and target. The observer returns current-state evidence only.

A useful target-matching round is therefore:

```text
critic gap
   ↓
MCP edit batch
   ↓
observer(requested camera/view)
   ↓
fresh evidence manifest + image
   ↓
outer critic
```

One fresh delivery render should be allowed to answer several related critic questions when appropriate. The observer should not force separate captures for composition, lighting and materials if the same render already contains the evidence.

## Implementation order

Build in this order and validate each stage before adding the next:

1. scoped state completeness flags;
2. camera render capture with provenance;
3. revision/checkpoint linkage and freshness states;
4. cache reuse and invalidation;
5. requested diagnostic views;
6. optional diffs/crops.

The v0.1 success criterion is practical: at least one previously necessary Computer Use screenshot can be replaced by a trustworthy observer capture while preserving the ability to detect the same visual defect.
