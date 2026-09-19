# AI Video Factory — Current Status

_Last updated: 2026-09-19_

## Current Verified Checkpoint

Current promoted GitHub workflow:

- **AI Shorts Automation Pipeline v2.32-Final**
- 68 nodes
- 66 connection source entries
- operational export has `pinData = {}`
- canonical GitHub workflow remains the known-good restore source

Canonical:

- `WORKFLOWS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final.json`

Archive:

- `BACKUPS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final_2026-09-19.json`

Protected older rollback baseline:

- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

Important:

- the locally validated node revisions listed below are **not yet promoted into the canonical workflow JSON**
- keep the canonical v2.32-Final workflow unchanged until downstream ComfyUI/media integration is validated

Detailed checkpoints:

- historical content/expansion checkpoint:
  - `DOCS/AI_VIDEO_FACTORY_AUTOMATION_CHECKPOINT_2026-09-19.md`
- latest visual/media follow-up checkpoint:
  - `DOCS/AI_VIDEO_FACTORY_VISUAL_MEDIA_CHECKPOINT_2026-09-19.md`

## Main Local LLM

Production baseline remains:

- **Qwen2.5 14B Instruct Q4_K_M**
- Runtime: Ollama

Qwen3.5 9B remains experimental only.

## Current Content Pipeline

```text
Pipeline Config
→ Prepare Run Folder
→ Concept Grounding Input
→ Concept Grounding - Ollama
→ Concept Grounding Validator
→ Evidence Query Builder
→ Evidence Query Splitter
→ SearXNG Search
→ Evidence Pack Builder
→ Fact Pack Input
→ Fact Pack - Ollama
→ Fact Pack Validator
→ Script Writer Input
→ Script Writer - Ollama
→ Script Validator
```

If the first Script Validator detects severe duration shortage:

```text
Script Validator
→ Need Evidence Expansion?
→ Evidence Expansion Query Builder
→ Evidence Expansion Query Splitter
→ SearXNG Search - Expansion
→ Evidence Expansion Pack Builder
→ Fact Pack Input Expanded
→ Fact Pack - Ollama Expanded
→ Fact Pack Validator Expanded
→ Script Writer Input Expanded
→ Script Writer - Ollama Expanded
→ Script Validator Expanded
→ Expanded Script Ready?
```

Evidence Expansion remains limited to **one pass only**.

If the expanded script still fails the minimum production threshold:

```text
needs_evidence_expansion = false
ready_for_scene_adapter = false
next_route = stop
→ Stop - Insufficient Verified Content
```

If the expanded script is acceptable:

```text
Expanded Script Ready?
→ Scene Adapter
→ Visual Prompt Planner - Ollama
→ Visual Prompt Validator
→ Scene Splitter
→ Prompt Engine v3.1
→ HTTP Request2
→ ComfyUI
```

## Latest Locally Verified Content Versions

Base chain:

- `Fact Pack Input`: **v1.8.4** — FROZEN
- `Fact Pack Validator`: **v1.6.4** — FROZEN
- `Script Writer Input`: **v1.5** — FROZEN
- `Script Validator`: **v1.8** — FROZEN

Expanded chain:

- `Evidence Expansion Pack Builder`: **v1.2**
- `Fact Pack Input Expanded`: **v1.8.5-expanded** — FROZEN
- `Fact Pack Validator Expanded`: **v1.6.5-expanded** — FROZEN
- `Script Writer Input Expanded`: **v1.5-expanded** — FROZEN
- `Script Validator Expanded`: **v1.8-expanded** — FROZEN

## Latest Runtime Test

Concept:

- `고양이가 갑자기 우다다하는 이유`

Target:

- 40 s
- 7-scene current runtime profile

Latest expanded script result:

- content integrity: PASS
- duration status: WARN
- estimated duration: **25.9 s**
- duration ratio: **0.647**
- `needs_evidence_expansion = false`
- `ready_for_scene_adapter = true`
- `next_route = scene_adapter`

The pipeline now proceeds to the visual route for this test instead of stopping after expansion.

## Evidence / Fact Policy

Current production rule remains:

- Facts must come from supplied Evidence.
- Exact support quotes are required.
- Do not infer unsupported cause, purpose, motivation, onset, duration, frequency, or certainty.
- One-source propositions remain `MEDIUM + QUALIFY`.
- Script Writer paraphrases validated Facts only.
- Do not create filler to hit duration.
- Preserve subject scope; kitten evidence must not be generalized to all cats.
- If verified information is insufficient, stop safely.

## Visual Chain — Current Local Verified State

### Scene Adapter

- `Scene Adapter v1.2.2` — **FROZEN**

Verified behavior:

- deterministic scene partition
- sentence-aligned semantic context
- subject-scope preservation
- subject + action + modifier must come from the same validated sentence
- no cross-sentence modifier transfer
- English ASCII-only image/motion prompts
- no direct Korean prompt copying
- abstract reason/uncertainty/duration wording excluded from visual prompts
- fixed-camera/simple-motion bias

### Visual Prompt Validator

- `Visual Prompt Validator v1.2` — **FROZEN**

Runtime-tested catches:

- Scene 4/6/7 invalid `cat + fast running` cross-sentence modifier mixing
- all 14 Korean prompt fields when Planner returned Korean instead of English ASCII

Final corrected seven-scene plan passed:

- structure
- ASCII
- technical fields/text
- editing-transition checks
- camera-motion checks
- unstable-motion checks
- subject scope
- semantic integrity
- modifier integrity
- proposition integrity
- warnings: none

### Scene Splitter

- **FROZEN**

Verified:

- exactly 7 items
- scene order 1→7
- narration/subtitle/prompt/runtime metadata preserved

### Prompt Engine v3.1

Current output validation: **PASS**

Per scene output includes:

- `image_positive_prompt`
- `image_negative_prompt`
- `video_positive_prompt`
- `video_negative_prompt`
- anchor/video filename prefixes
- `workflow_profile = FLUX_LTX_I2V_v01_Success`

Current semantic plan:

```text
Scene 1 → cat + idle
Scene 2 → kitten + running
Scene 3 → kitten + running + pouncing
Scene 4 → cat + running
Scene 5 → cat + running
Scene 6 → cat + running
Scene 7 → cat + running
```

No invalid fast modifier remains in Scene 4/6/7.

## Stable Media Baseline

- FLUX anchor image
- LTX 0.9.6 I2V
- 416×736
- 121 frames per scene
- 20 fps output
- 30 fps LTX conditioning
- approximately 6.05 s per generated scene
- RIFE OFF
- fixed-camera/simple-motion bias
- FFmpeg final assembly

Scene count remains runtime-driven.

## Current Main Blocker

Current immediate blocker is **ComfyUI connectivity**, not prompt validation.

`HTTP Request2` returned:

```text
ECONNREFUSED 192.168.65.254:8188
```

The user confirmed ComfyUI was not connected for that run.

Current interpretation:

- not a Prompt Engine JSON failure
- not a visual prompt validation failure
- ComfyUI must be started/connected on port 8188 with Docker/n8n access before retrying

## Immediate Next Step

1. start/connect ComfyUI
2. verify local ComfyUI access on port 8188
3. rerun `HTTP Request2`
4. confirm one `prompt_id` per expected scene
5. verify ComfyUI History
6. verify actual FLUX anchor and LTX video file creation
7. continue downstream polling/mapping/scene-collection validation

Do not modify the now-verified visual-content nodes merely because ComfyUI was offline.

## Deferred Issues

### Search resilience

Still relevant but deferred from the current immediate media step:

- Brave rate limiting
- DuckDuckGo CAPTCHA
- Wikidata timeout/rate-limit behavior

Do not weaken evidence rules to compensate for upstream search instability.

### TTS metadata/runtime alignment

Still deferred:

- workflow metadata contains older CosyVoice3 references
- recent local runtime operation used an Edge-TTS-compatible path/container

Reconcile later as a separate verified change.

## Development Rule

Follow `PROJECT_RULES/DEVELOPMENT_POLICY.md`.

- one verified node/stage at a time
- full revised Code/config for node changes
- preserve known-good nodes
- deterministic logic before extra LLM calls
- local-first / free-first
- evidence quality and factual safety before target-duration padding
- preserve rollback points before promotion

## GitHub Promotion Boundary

Do **not** overwrite the canonical v2.32-Final workflow yet.

Promote the current local workflow only after:

1. ComfyUI connectivity succeeds
2. HTTP Request2 returns valid prompt IDs
3. expected anchor/video files are physically created
4. downstream media integration is validated sufficiently to rule out regressions
5. the updated local n8n workflow is exported and checked
