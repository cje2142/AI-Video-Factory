# AI Video Factory
## n8n Pipeline Redesign Spec v1.0

**Status:** DESIGN LOCKED / PRE-IMPLEMENTATION BASELINE  
**Purpose:** Implementation specification for migrating the verified v0.71 full-automation baseline from AnimateDiff T2V to FLUX Anchor → LTX I2V while preserving proven downstream automation.

---

## 1. Rollback Baseline

Official rollback point:

`WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

v0.71 is preserved as the first verified end-to-end automation milestone from planning through final MP4. Visual generation quality is not considered final.

---

## 2. Final Logical Architecture

```text
BLOCK A — Input / Config
Manual Trigger / Input
↓
Pipeline Config
↓
Prepare Run Folder

BLOCK B — Planning
Qwen Production Planner
↓
Plan Validator
↓
Production Runtime
↓
Scene Splitter

BLOCK C — Scene Intelligence
Scene Analyzer
  ├─ Scene Type
  ├─ Risk
  └─ Motion Level
↓
Prompt Engine
  ├─ FLUX Prompt
  └─ LTX Motion Prompt

BLOCK D — Generation
FLUX Submit
↓
FLUX History Tracker
↓
Anchor Check
↓
LTX I2V Submit
↓
LTX History Tracker
↓
Visual Validator
↓
Retry Controller
↓
Fallback Gate

BLOCK E — Scene Collection
Scene Collection
↓
Download Scene MP4
↓
Restore Video Metadata
↓
Save Scene MP4
↓
Create FFmpeg List
↓
Scene Merge
↓
Video Ready

BLOCK F — Audio / Subtitle
Scene Narrations
↓
TTS
↓
Scene MP3
↓
Continuous Narration
↓
Whisper
↓
SRT
↓
BGM Selection (optional, local-first)

BLOCK G — Final
Video Ready
Narration Ready
Subtitle Ready
BGM Ready (optional)
↓
Final Asset Sync
↓
FFmpeg Final Compose
↓
Final AV Validator
↓
Run Manifest
↓
shorts_final.mp4
```

---

## 3. Pipeline Config Principle

Separate desired targets from actual production results.

### Pipeline Config = target values

Contains:

```text
topic
concept
target
tone
visual_style
bgm_direction

target_duration
scene_duration_target
scene_duration_min
scene_duration_max

default_engine = FLUX_LTX_I2V
fallback_engine = WAN_I2V
retry_limit = 1
rife_enabled = false

TTS
Whisper
Subtitle Style
Output
Paths
Archive
```

Remove fixed production assumptions such as:

```text
scenes = 10
total_duration = 40
frames_per_scene = 16
fps = 4
DreamShaper checkpoint
AnimateDiff motion model
FreeInit
AnimateDiff context settings
```

### Current LTX baseline profile

```text
resolution: 416×736
frames: 97
frame_rate: 30
steps: 8
cfg: 1.0
strength: 1.0
sampler: Euler
RIFE: OFF
```

---

## 4. Production Runtime

`Production Runtime` is a NEW deterministic Code Node placed after Plan Validator.

Role:

```text
Pipeline Config = desired target
Production Runtime = actual production values
```

Runtime owns:

```text
actual_scene_count
expected_scene_count
actual_total_duration
scene durations
run_id
run_name
pipeline_version
```

All downstream fixed references such as `config.video.scenes` must be replaced by Runtime values.

---

## 5. Qwen Production Planner

Planner handles content meaning only.

```text
User Intent
→ Core Intent
→ Story Plan
→ Narration
→ Scene Plan
→ BGM Direction
```

Planner must NOT output FLUX/LTX technical parameters or generation prompts.

Required Scene structure:

```json
{
  "scene": 1,
  "purpose": "",
  "narration": "",
  "subtitle": "",
  "subject": "",
  "action": "",
  "environment": "",
  "framing": "",
  "camera": "",
  "continuity": "",
  "duration": 5
}
```

Remove:

```text
investment-domain hard-coding
fixed 10-scene story
fixed scene-ending patterns
investment-principle ending requirement
factory/server/business visual bias
```

Preserve:

```text
logical scene continuity
one useful point per scene
natural Korean narration
short subtitles
no unsupported factual invention
visual relevance to narration
avoid repetitive locations/compositions
```

---

## 6. Plan Validator

Deterministic validation only where possible.

Validate:

```text
valid JSON
required root fields
scene array exists
scene numbering is sequential and unique
required Scene fields are present
duration is within configured range
total duration is within acceptable target range
no abnormal scene count
no technical parameters inside Planner output
```

Forbidden Planner technical fields include:

```text
width
height
frames
fps
steps
cfg
strength
sampler
scheduler
checkpoint
model_name
seed
flux_seed
ltx_seed
```

---

## 7. Scene Splitter

Scene Splitter consumes validated Plan + Production Runtime.

Each Scene item carries:

```text
scene
scene_count
duration
purpose
narration
subtitle
subject
action
environment
framing
camera
continuity
run_id
run_name
pipeline_version
```

No fixed scene count is allowed.

---

## 8. Scene Analyzer

Scene Analyzer does not redesign the story.

It determines only:

```text
Scene Type
Risk
Motion Level
```

Allowed Scene Types:

```text
DRONE_NATURE
CHARACTER_3D
SCREEN_UI
PRODUCT_OBJECT
REAL_HUMAN
```

Risk output:

```json
{
  "human": "NONE|LOW|HIGH",
  "detail": "LOW|MEDIUM|HIGH",
  "motion": "LOW|MEDIUM|HIGH"
}
```

Motion levels:

```text
LOW
MEDIUM
HIGH
```

Core rule:

```text
Risk ↑
→ Motion ↓
→ Shot simpler
→ Prompt simpler
```

Initial implementation should use a deterministic Code Node rather than a separate Qwen call.

---

## 9. Prompt Engine

Replace existing AnimateDiff T2V Prompt Engine.

Input:

```text
Scene Plan
+ Scene Type
+ Risk
+ Motion Level
```

Output:

```text
FLUX Prompt
FLUX Negative Prompt
LTX Motion Prompt
LTX Negative Prompt
```

### FLUX role

```text
Subject
Composition
Pose
Environment
Lighting
Geometry
```

FLUX should stabilize the first frame and avoid unnecessary motion language.

### LTX role

```text
Preserve Anchor
Add restrained motion
Add viewpoint/perspective change when appropriate
Maintain continuity
```

For REAL_HUMAN, default to low motion and stable hands/body. Current stable pattern emphasizes slow continuous motion, steady gaze, minimal eye movement, and avoids fast/jerky gestures.

For PRODUCT_OBJECT, prefer `viewpoint shift`, `perspective change`, and `parallax` rather than broad `camera motion` wording.

---

## 10. FLUX Anchor Generation

Generation path:

```text
Prompt Engine
→ FLUX Submit
→ FLUX History Tracker
→ Anchor Check
```

Store anchors under:

`/files/runs/{run_id}/anchors/`

Recommended naming:

`scene_01_anchor.png`

---

## 11. Anchor Check

Initial deterministic checks only:

```text
file exists
file decodes
expected resolution
minimum file size
valid scene/path metadata
```

If Anchor fails:

```text
regenerate FLUX only
```

Do not initially add a heavy AI Vision validator.

---

## 12. LTX I2V Generation

Path:

```text
Valid FLUX Anchor
+ LTX Motion Prompt
→ LTX I2V
→ LTX History Tracker
```

If Anchor is valid and LTX fails, do not regenerate FLUX automatically.

---

## 13. Visual Validator / Retry / Fallback

Validation states:

```text
PASS
MINOR
CRITICAL
```

CRITICAL examples:

```text
severe face/hand/body deformation
major frame-to-frame structure collapse
strong flicker
object morphing
major anchor identity/composition drift
scene meaning failure
```

Retry rule:

```text
CRITICAL
→ same FLUX Anchor
→ new LTX seed
→ retry once
```

If CRITICAL again:

```text
Fallback Gate
→ mark fallback_required = true
```

Wan I2V remains a reserved fallback interface only until the FLUX → LTX main path is stable.

No unlimited regeneration.

---

## 14. Scene Collection

Reuse the verified v0.71 History / Download / Save / FFmpeg infrastructure.

Replace fixed scene-count logic with Runtime values.

Each collected Scene should retain:

```text
scene
prompt_id
filename
validation_status
retry_count
flux_seed
ltx_seed
run_id
```

---

## 15. TTS / Whisper / SRT / BGM

Reuse the verified v0.71 audio/subtitle pipeline as much as possible.

```text
Scene Narrations
→ TTS
→ Scene MP3
→ Continuous Narration
→ Whisper
→ SRT
```

Replace fixed duration/count references with Production Runtime values.

BGM policy:

```text
optional
local-first
Planner outputs mood/style/tempo only
n8n selects from local BGM library
```

Do not add music-generation complexity in this stage.

---

## 16. Final FFmpeg Compose

Inputs:

```text
Scene Video
Narration Audio
SRT
BGM (optional)
```

FFmpeg remains a deterministic compositor only.

Use `runtime.actual_total_duration` rather than fixed config duration.

---

## 17. Final AV Validator

Use ffprobe or equivalent deterministic checks.

Validate:

```text
final file exists
video stream exists
audio stream exists
expected duration range
audio/video duration difference
subtitle exists
scene count complete
file size valid
file decodes
output path valid
```

Final success state:

`FINAL_PASS`

---

## 18. Run Manifest

Create one manifest per run.

Minimum structure:

```json
{
  "run_id": "",
  "scene_count": 0,
  "scenes": [
    {
      "scene": 1,
      "flux_seed": 0,
      "ltx_seed": 0,
      "retry_count": 0,
      "validation": "PASS",
      "output": ""
    }
  ],
  "final_validation": "FINAL_PASS",
  "final_file": "shorts_final.mp4"
}
```

Purpose:

```text
reproducibility
debugging
failure-rate tracking
retry tracking
future fallback analysis
```

---

## 19. Path Layout

Recommended Run layout:

```text
/files/runs/{run_id}/
├─ anchors/
│  └─ scene_XX_anchor.png
├─ scenes/
│  └─ scene_XX.mp4
├─ tts_audio/
├─ subtitle.srt
├─ final_narration.mp3
├─ final_output.mp4
├─ shorts_final.mp4
└─ manifest.json
```

---

## 20. v0.71 Reuse / Replacement Policy

### Keep or minimally modify

```text
Manual Trigger
Prepare Run Folder
Wait / polling cadence
ComfyUI /history access pattern
If ready gate
Scene MP4 download
file saving
Scene FFmpeg merge
TTS Engine
Whisper
SRT generation
asset sync
Final FFmpeg framework
Run ID / path management
```

### Modify

```text
Pipeline Config
Qwen Production Planner
Plan Validator
Scene Splitter
History metadata mapping
Scene count checks
Runtime duration checks
TTS metadata restoration
Video/Narration/Subtitle Ready metadata
Final file validation
```

### Replace

```text
AnimateDiff T2V Prompt Engine
AnimateDiff ComfyUI generation request
```

### New

```text
Production Runtime
Scene Analyzer
FLUX Prompt Builder
FLUX Submit / History handling
Anchor Check
LTX Prompt Builder
LTX I2V Submit / History handling
Visual Validator
Retry Controller
Fallback Gate
Final AV Validator
Run Manifest
```

---

## 21. Implementation Order

Implement one block at a time and preserve rollback after each verified stage.

```text
1. Pipeline Config
2. Qwen Production Planner
3. Plan Validator
4. Production Runtime
5. Scene Splitter
6. Scene Analyzer
7. Prompt Engine
8. FLUX generation + Anchor Check
9. LTX I2V generation
10. Visual Validator + Retry
11. Scene Collection adaptation
12. TTS / Whisper / SRT Runtime adaptation
13. BGM optional path
14. Final FFmpeg
15. Final AV Validator
16. Run Manifest
17. Full end-to-end verification
```

Do not promote this spec to FINAL implementation status until the redesigned workflow completes verified end-to-end execution.

---

## 22. Design Principles

```text
Visual Stability > Story/Concept Continuity > Natural Motion > Prompt Following > Fine Detail
```

Operational principles:

```text
local-first
free-first
minimal engine count
no unnecessary multi-service routing
no fixed scene count
controlled retry only
no unlimited regeneration
preserve proven v0.71 downstream assets
separate creative planning from generation mechanics
separate target Config from actual Runtime
human final decision remains authoritative
```

---

## 23. Design Lock Decision

The redesign architecture is sufficiently validated for implementation.

Next phase is execution, not further architecture expansion.

Any design change discovered during implementation must be documented, justified by actual test evidence, and validated before being merged into the locked specification.
