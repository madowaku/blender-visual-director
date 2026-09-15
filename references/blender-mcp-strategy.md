# BlenderMCP strategy and capability audit

## Verified scope

Audited 2026-09-15: [ahujasid/blender-mcp at 7684c6b](https://github.com/ahujasid/blender-mcp/tree/7684c6b3ad2aa0710bbdb1cb06b497c90899ae00), package **1.9.4**. An isolated installation returned **31 tools** through real MCP `initialize` and `tools/list`. The MCP SDK identified itself as 1.30.0 in `serverInfo.version`; that is not the BlenderMCP package version. The host session had no registered BlenderMCP connector. See [evaluation.md](evaluation.md) for execution evidence, separate from schema discovery.

Inspect the installed server's schema before use. Client namespace prefixes and versions vary. The following are verified tool names without a client prefix; `Context` is injected by the server and is not an argument to send.

| Need | Verified public tool and relevant inputs | Result / constraint |
| --- | --- | --- |
| Version / connection | `get_addon_status(user_prompt="")` | JSON text: Blender/addon/protocol versions, capabilities and warnings; error text on failure |
| Scene summary | `get_scene_info(user_prompt)` | Required `user_prompt`; scene name, total object count, material count, **first 10** objects with name/type/rounded local location |
| One object | `get_object_info(object_name, user_prompt="")` | Local transforms, view-layer visibility, material names; mesh counts and world bounding box for meshes |
| Edit or custom query | `execute_blender_code(code, user_prompt="")` | Executes Python; returns captured stdout in a text envelope, not an image or automatic JSON object |
| Viewport image | `get_viewport_screenshot(max_size=1000, user_prompt="")` | MCP PNG Image; current first `VIEW_3D` area, not a requested angle or final render. Schema default is 1000 despite an older 800 value in its prose |
| Node schema | `describe_node_type(bl_idname, property_overrides=null, user_prompt="")` | Property and socket data; use if available before unfamiliar node inputs/enums |
| Blender API query | `bpy_api_lookup(query, user_prompt="")` | Runtime RNA/operator information; query the needed type/property rather than guessing |
| Export | `export_scene(filepath, format="glb", object_names=null, selection_only=false, apply_modifiers=true, user_prompt="")` | GLB/FBX export, not rendering or `.blend` saving |

Supply `user_prompt` according to the installed schema and description; do not replace the user's intent with an invented goal. Inspect text payloads for errors even when the MCP transport reports success.

Other discovered groups were Poly Haven, Sketchfab and Poly Pizza search/download; Hyper3D and Hunyuan3D generation/import/status; telemetry controls and trajectory feedback. They are optional integrations, not prerequisites for this skill. Do not turn an ordinary modeling task into an asset-service setup task.


Live execution also found a source-build packaging error in `get_addon_status` (`No module named 'blender_mcp.config'`) with `isError=false`. If a status tool fails, inspect its text and use an available read-only Python query for `bpy.app.version_string` and scene identity; do not equate a status-helper error with a dead edit connection. Do not patch or disable protections as a routine production workaround.

## Five-way classification

### 1. Operations possible through BlenderMCP

Through the verified Python execution tool: create and revise geometry, transforms, modifiers, materials and nodes; configure lights/cameras; save `.blend`, render to an authorized path, and use Blender import/export operators. Batch related operations with their own preconditions. These are Python capabilities through one tool, not dedicated MCP endpoints. Asset acquisition and GLB/FBX export also have public tools as above, subject to integration availability.

### 2. Structured scene / object information

The standard scene query is a small summary, not a complete inventory. Compare `object_count` with returned entries; absence after the tenth entry is not deletion. Object mesh counts describe source data and do not prove evaluated modifier output is correct. Local transforms are not world transforms; `visible_get()` does not establish render visibility. Neither query fully describes shader graphs, lighting, camera framing, collections or occlusion.

### 3. Images / viewport / render information

The screenshot tool returns actual image content. At the audited revision the addon first tries GPU offscreen `draw_view3d`, then falls back to a window-area grab. It requires an appropriate viewport and GPU/UI context. This is not guaranteed headless rendering. The addon reports capture method internally, but the public image tool discards that metadata. It also depends on a temporary path readable by both processes; remote/container setups may fail without shared storage.

There is **no dedicated final-render image retrieval tool** in the audited list. Python can render/save via Blender operators, but a returned filepath is not a viewed image. Retrieve it through an available file-image tool/shared storage; otherwise select Computer Use to inspect Render Result when feasible. Sketchfab previews show external assets, not the current scene.

### 4. Information currently requiring Computer Use for direct observation

The actual displayed Blender window, OS-owned dialogs, splash/modal overlays, focus/occlusion, and UI areas outside the captured 3D viewport are not directly observed by those public tools. Use the available Computer Use implementation when that display state matters. Many internal UI settings are Python-readable, so this is a gap in direct pixel observation, not a claim that all UI state is inaccessible to Python. Silhouette, lighting and composition can be judged from suitable MCP images; they are not inherently GUI-only.

### 5. Low-cost additions through Blender Python

Return selected JSON fields with `print(json.dumps(payload))`: complete scoped object names and counts; matrices and world bounds; dimensions; parent/collection membership; modifier types and visibility; material assignments and selected shader values; camera transform/projection/clip settings; lights; render engine/resolution/frame/color management; missing resource paths. Update the view layer before reading changed transforms.

Compute evaluated mesh counts, non-manifold edges, degenerate faces, or camera-projected bounds only when the task needs them; large meshes make these less cheap. Release temporary evaluated meshes with `to_mesh_clear()`. Topology warnings and projected bounds are diagnostics, not proof of visual failure/success: open meshes may be intentional and projected bounds do not establish occlusion.

## Batch and recovery choices

- Group changes by one outcome (e.g. robot torso and limb proportions), not one call per primitive or a whole unbounded production script. Put tightly scoped postconditions in the same call when useful; the next decision still follows inspection of the returned state.
- Use known names or scoped identifiers, direct data access where possible, and explicit operator context. Query version-dependent properties/enums; use node types rather than localized display names.
- A Python batch is **not atomic**. After an exception or timeout, inspect current state before retrying. Repair only the failed outcome and avoid duplicate objects/materials. Save a recoverable checkpoint before costly or destructive batches within the user's task.
- Respect safe mode. The audited server permits normal `bpy` rendering/saving and pure Python JSON output, but rejects direct file `open`, OS/subprocess/network access, and persistent handlers/timers. Let the client persist returned JSON when needed; do not disable safe mode to manufacture an Evidence Bundle.
- Upstream prose recommends screenshots after every change. For this user's checkpoint workflow, treat a coherent batch as the unit of production and schedule images by visual risk; do not mechanically replay per-operation screenshots. This does not waive requirements imposed by higher-priority instructions or the active UI tool's input rules.
- If an image is black, stale, misframed or in the wrong shading mode, fix the known view/capture cause through MCP and retry once. If the cause is unknown, switch directly to a suitable render/image channel or targeted UI observation. If still inadequate, report the missing evidence; do not loop on identical screenshots.
- If MCP is missing/disconnected, do available read-only discovery and report the concrete connection requirement. A narrowly scoped GUI recovery can help an already authorized setup; the skill itself does not authorize connector installation or broad GUI replacement. v0.1 provides no server/addon implementation.

Implementation evidence: [server tools](https://github.com/ahujasid/blender-mcp/blob/7684c6b3ad2aa0710bbdb1cb06b497c90899ae00/src/blender_mcp/server.py), [addon queries and capture](https://github.com/ahujasid/blender-mcp/blob/7684c6b3ad2aa0710bbdb1cb06b497c90899ae00/addon.py). These links establish provenance; runtime discovery remains authoritative for availability.
