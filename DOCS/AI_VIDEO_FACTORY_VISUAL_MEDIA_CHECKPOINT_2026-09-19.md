# AI Video Factory — Visual/Media Pipeline Follow-up Checkpoint

_Date: 2026-09-19_
_Status: local visual-content chain validated through Prompt Engine; ComfyUI connection pending_

## 1. Scope

This follow-up checkpoint records progress after the earlier 2026-09-19 automation checkpoint.

The canonical GitHub workflow remains unchanged:

- `WORKFLOWS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final.json`
- canonical workflow is still the known-good GitHub restore source
- local node revisions described below have **not yet been promoted into the canonical workflow JSON**

Protected rollback baseline remains:

- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

The earlier historical checkpoint remains preserved:

- `DOCS/AI_VIDEO_FACTORY_AUTOMATION_CHECKPOINT_2026-09-19.md`

## 2. Runtime Test Topic

Concept:

```text
고양이가 갑자기 우다다하는 이유
```

Target:

- 40 s
- 7-scene current runtime profile

The expanded content route no longer ends at the previous safe STOP for this test. After local node fixes and re-validation, the expanded script was allowed to continue to Scene Adapter.

Latest expanded script result:

- content integrity: PASS
- duration status: WARN
- estimated duration: 25.9 s
- target duration: 40 s
- duration ratio: 0.647
- `needs_evidence_expansion = false`
- `ready_for_scene_adapter = true`
- `next_route = scene_adapter`

This is intentionally a conservative factual script rather than padded narration.

## 3. Current Locally Verified Content Node Versions

Base chain:

- `Fact Pack Input` → **v1.8.4** — FROZEN
- `Fact Pack Validator` → **v1.6.4** — FROZEN
- `Script Writer Input` → **v1.5** — FROZEN
- `Script Validator` → **v1.8** — FROZEN

Expanded chain:

- `Evidence Expansion Pack Builder` → **v1.2**
- `Fact Pack Input Expanded` → **v1.8.5-expanded** — FROZEN
- `Fact Pack Validator Expanded` → **v1.6.5-expanded** — FROZEN
- `Script Writer Input Expanded` → **v1.5-expanded** — FROZEN
- `Script Validator Expanded` → **v1.8-expanded** — FROZEN

Important evidence behavior retained:

- exact support quote boundary
- one-source proposition remains `MEDIUM + QUALIFY`
- subject scope preserved
- kitten evidence is not generalized to all cats
- unsupported cause, purpose, motivation, physiology, or stronger certainty is blocked
- no filler is invented to hit target duration

## 4. Scene Adapter / Visual Planner Validation

### Scene Adapter v1.2.2 — FROZEN

Current local Scene Adapter:

- keeps deterministic scene partition
- keeps sentence-aligned `semantic_context`
- hides topic/concept metadata from the Visual Planner factual boundary
- preserves subject scope:
  - `새끼 고양이` → kitten
  - `고양이` → cat
- requires subject + action + modifier to be supported by the **same validated sentence**
- forbids cross-sentence modifier transfer
- requires `image_prompt` and `motion_prompt` to be **English ASCII only**
- prevents copying Korean narration directly into generation prompts
- excludes abstract/non-visual narration such as reason/uncertainty/duration wording from visual prompts
- fixed-camera/simple-motion policy retained

### Visual Prompt Validator v1.2 — FROZEN

Validator v1.2 was tested against real failure cases.

Verified catches:

1. **Modifier integrity**
   - rejected Scene 4 / 6 / 7 when `cat + fast running` was assembled by mixing:
     - `cat + running` from one sentence
     - `fast` from a different kitten sentence

2. **ASCII enforcement**
   - rejected all 14 Korean prompt fields when the Planner returned Korean:
     - 7 `image_prompt`
     - 7 `motion_prompt`

3. **Final corrected Planner output**
   - all seven scenes PASS
   - structure PASS
   - ASCII PASS
   - technical fields/text PASS
   - editing transition PASS
   - camera motion PASS
   - unstable motion PASS
   - subject scope PASS
   - semantic integrity PASS
   - modifier integrity PASS
   - proposition integrity PASS
   - warnings: none

Final semantic pattern:

```text
Scene 1 → cat + idle
Scene 2 → kitten + running
Scene 3 → kitten + running + pouncing
Scene 4 → cat + running
Scene 5 → cat + running
Scene 6 → cat + running
Scene 7 → cat + running
```

No invalid `fast` transfer remains in Scene 4/6/7.

## 5. Scene Splitter

`Scene Splitter` is locally verified and treated as FROZEN.

Observed result:

- exactly 7 output items
- scene order 1→7 preserved
- each item retained:
  - `scene`
  - `narration`
  - `subtitle`
  - `image_prompt`
  - `motion_prompt`
  - `title`
  - `script_summary`
  - `bgm_direction`
  - `run_id`
  - `run_name`
  - `pipeline_version`

## 6. Prompt Engine v3.1

`Prompt Engine v3.1` passed current structural/semantic validation.

Verified output per scene:

- `image_positive_prompt`
- `image_negative_prompt`
- `video_positive_prompt`
- `video_negative_prompt`
- `anchor_filename_prefix`
- `video_filename_prefix`
- `workflow_profile = FLUX_LTX_I2V_v01_Success`

Meaning preservation remained intact:

- kitten/cat scope preserved
- Scene 3 keeps running + pouncing
- Scene 4→7 keep cat + running
- invalid fast modifier did not reappear
- fixed-camera/stable-motion instructions retained

Quality observation, not a current failure:

- common LTX positive prompt adds `Motion is slow, smooth, continuous...`
- this may reduce the visible speed impression of zoomies
- leave unchanged until actual generated-video evidence justifies a media-profile change

## 7. Current Media Blocker

Next node:

```text
Prompt Engine v3.1
→ HTTP Request2
→ ComfyUI
```

Current `HTTP Request2` failure:

```text
ECONNREFUSED 192.168.65.254:8188
```

Interpretation:

- request reached the Docker host address resolution layer
- connection was refused because ComfyUI was not connected/running for this test
- this is **not currently treated as a Prompt Engine JSON or prompt-content failure**

The user confirmed that ComfyUI had not yet been connected.

## 8. Immediate Next Step

1. start/connect ComfyUI on port 8188 with Docker/n8n access
2. verify `http://127.0.0.1:8188` locally
3. rerun `HTTP Request2`
4. confirm one `prompt_id` per expected scene
5. verify ComfyUI History and actual anchor/video file creation
6. continue downstream polling / mapping / scene collection validation

Do not modify Prompt Engine or visual-content nodes merely because ComfyUI was offline.

## 9. Deferred Issues

Still deferred as isolated future work:

### Search reliability

Observed SearXNG upstream instability remains relevant:

- Brave rate limiting
- DuckDuckGo CAPTCHA
- Wikidata timeout/rate-limit behavior

Do not weaken evidence rules to compensate for upstream search instability.

### TTS metadata/runtime alignment

Still deferred:

- workflow metadata contains older CosyVoice3 references
- recent local runtime operation used an Edge-TTS-compatible path/container

Reconcile later as a separate verified change.

## 10. Promotion Rule

Do **not** overwrite the canonical v2.32-Final workflow yet.

Promotion should happen only after:

1. ComfyUI connection succeeds
2. HTTP Request2 returns valid prompt IDs
3. expected FLUX anchors and LTX videos are physically created
4. downstream media route is validated far enough to rule out integration regressions
5. updated local workflow is exported and checked before GitHub promotion

Until then:

- canonical GitHub workflow = rollback source
- local workflow = active validation branch/state
