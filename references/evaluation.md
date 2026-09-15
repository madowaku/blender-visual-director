# Evaluation: quality and observation cost

## Reusable A/B/C protocol

Use a small task such as a stylized robot with specified colors, proportions and camera composition. Start each condition from the same scene and brief, with matching Blender/MCP versions, asset access, model settings, resolution, lighting budget and stopping criteria.

- **A:** BlenderMCP without this skill; retain its normal guidance. MCP screenshots and Python renders are allowed. This is not an image-blind baseline.
- **B:** BlenderMCP plus this skill; review MCP images or readable render files, without Computer Use.
- **C:** BlenderMCP plus this skill and a selected Computer Use checkpoint. For an explicit comparison measure the extra sensor once; ordinary production still requires a reason to use it.

Keep each run's context separate when possible. Label shared-agent or prescribed-policy pilots accurately: they demonstrate mechanics/cost, not the skill's causal effect. Disclose shared fixes. If an arm is unavailable, report **not run** and the missing capability, not zero cost or quality.

## Metrics

| Metric | Counting rule |
| --- | --- |
| Completion quality | Same delivery view and rubric across arms: silhouette, proportion, shape language, balance, readability, composition, material separation, lighting, obvious artifacts. Give evidence-based judgments; mark unseen criteria unknown |
| MCP calls | Every tools/call request, including failures/retries. Report initialization/discovery separately; count internal addon requests separately if measured |
| Computer Use operations | Each Computer Use API operation, divided into discovery, observations and GUI inputs. Also count tool invocations if multiple APIs share one invocation |
| Images / visual reviews | Separate generated/captured images, images actually inspected, and review events. A contact sheet does not make several captures count as one |
| Rework | Correction batches after a failed check, separate from planned production and setup recovery |
| Obvious failures | Symptom, detection, resolution; retain failed captures in totals |
| Context / tokens | Actual usage if exposed; otherwise text bytes/chars and image counts/resolutions as proxies. State inclusion of prompts, code, schemas and skill references. Do not infer exact image tokens from file size |

Separate setup from production. Internal trajectory snapshots or images are different from requested/agent-viewed evidence; report unknown internal counts honestly. Do not change user telemetry preferences to improve metrics.

Prefer blinded review for stronger claims. Repeat with silhouette revisions, lighting failure, camera crop, more than ten objects and unavailable images before claiming general savings.

## v0.1 measured pilot: 2026-09-15

Environment: Blender **5.2.1 LTS**, upstream BlenderMCP **1.9.4 / 7684c6b**, MCP SDK 1.30.0, Windows. A temporary stock addon ran in a factory-startup test process; a temporary SDK client made real MCP calls. No Codex connector registration or permanent addon installation was performed. The test client requested telemetry disabled and safe mode enabled. No custom server/addon ships in the skill.

Brief: "Create a small teal toy robot with cream face and orange hands, clear silhouette, balanced proportions, and a three-quarter studio composition."

Both arms used identical geometry/material/camera/lighting code in separate scenes, EEVEE, 640 x 640 PNG, AgX. A used core / limbs / finish batches, each followed by standard scene information and viewport capture. B combined core and limbs, returned scoped state from its edits, and captured blockout plus camera renders. Both received the same correction for disconnected shoulders/neck and floating feet.

This was a **shared-agent, prescribed-policy feasibility pilot**, not independent skill-on/skill-off agents. A's core and limb images were inspected together after a file-viewer infrastructure failure; corrected outputs were also inspected together. The correction diagnosis was shared. Counts demonstrate executed routes, not a causal quality or token-saving claim.

| Production metric | A: MCP baseline policy | B: MCP + director policy | C: director + CU |
| --- | --- | --- | --- |
| MCP tools/call requests | 10 | 4 | Not run |
| Planned edit batches / correction batches | 3 / 1 | 2 / 1 | Not run |
| MCP viewport images | 3 | 1 | Not run |
| Camera renders generated | 2 | 2 | Not run |
| Images actually inspected | 5 | 3 | Not run |
| Review events (can include multiple images) | 3 | 3 | Not run |
| Successful CU operations / GUI inputs | 0 / 0 | 0 / 0 | Not run |
| MCP production errors | 0 | 0 | Not run |
| Tool argument characters, including code/prompt | 10,965 | 8,926 | Not run |
| Returned text characters | 10,806 | 6,620 | Not run |

C's Computer Use import timed out on initialization and one retry. No window discovery, screenshot or GUI input was reached: **two failed setup invocations**, not a successful zero-CU run. An earlier node-based file viewer also timed out. Saved MCP/render images were inspected through a local file-image display fallback. Neither infrastructure failure is a Blender production call.

**Quality:** corrected A/B render pixels were identical. The reviewed camera showed readable eyes/face/hands, consistent beveled forms, balanced stance, usable framing/lighting and distinct teal/cream/orange/dark surfaces. Initial shoulder/neck disconnection and foot ground gaps were corrected in one MCP batch per arm and visually rechecked. Hidden/back geometry and animation were not assessed. This is a small toy-scene feasibility result, not a production-quality benchmark.

**Cost:** requested image count fell from 5 to 3 (40%); MCP calls from 10 to 4 (60%); actual review events stayed at 3. Exact total tokens were unavailable. Argument/response characters exclude schemas, discovery, reasoning and skill instructions. The 4,392-character entrypoint plus strategy (about 9.2k) and visual-review (about 4.1k) references can outweigh savings on a tiny first task; total context reduction is **not established**.

### Additional execution findings

- Real MCP tools/list returned 31 public tools. Live scene, node-schema, Python editing, viewport PNG, Blender save and camera-render paths worked.
- Standard scene information returned 10 of 13 objects, then 10 of 18. Scoped Python returned the complete inventory, finally all 21 objects.
- get_addon_status returned error text about missing blender_mcp.config with MCP isError=false. Python returned the actual Blender version and other tested tools worked. This source-build packaging error is distinct from a dead edit connection.
- An independent decision simulation handled a 24-object truncated summary, stale camera image, black successful capture and missing-MCP branch. It made no live edits or invented results. Its small wording improvement (switch channel immediately if a black capture has no known cause) was incorporated.

The local development audit retains tools-list JSON, per-call JSONL, fixed pilot code, PNGs, .blend files and the forward-test report. These are development evidence, not runtime skill resources. No test harness or image gallery is required to load the skill.

## Recommended v0.2 evaluation

Restore working Computer Use and run C against the same brief. Then use independent contexts, the same starting .blend, blinded review and a shared stopping criterion. Include an intentional crop or lighting defect plus failed/stale viewport retrieval, so the extra sensor can demonstrate a benefit instead of merely adding a screenshot. Separate cold skill-loading cost from subsequent edits.
