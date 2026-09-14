# AI Video Factory
## n8n Pipeline Redesign Spec v1.1

**Status:** DESIGN LOCKED / PRE-IMPLEMENTATION BASELINE  
**Purpose:** Implementation specification for migrating the verified v0.71 full-automation baseline from AnimateDiff T2V to FLUX Anchor → LTX I2V, while preserving proven downstream automation and adding the finalized local Audio/Subtitle/BGM tool stack and GPU memory policy.

---

## 1. Rollback Baseline

Official rollback point:

`WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

v0.71 is preserved as the first verified end-to-end automation milestone from planning through final MP4. Visual generation quality is not considered final.

v1.0 redesign spec remains preserved unchanged. v1.1 extends it with the finalized Audio/Subtitle/BGM tool stack and Stage-based VRAM policy.

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

BLOCK D — Visual Generation
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
↓
Visual Stage Complete
↓
GPU Memory Release

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

BLOCK F — Narration / Subtitle
Scene Narrations
↓
CosyVoice 3
↓
Scene Audio
↓
faster-whisper QC (optional but recommended)
↓
Continuous Narration
↓
Original Subtitle Text + Timing
↓
SRT Engine
↓
Subtitle Ready

Advanced optional:
WhisperX word-level alignment

BLOCK G — BGM
Planner BGM Direction
↓
Local BGM Library (default)
↓
ACE-Step 1.5 (optional generation only when needed)
↓
BGM Ready

BLOCK H — Final
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

TTS profile
TTS QC profile
Subtitle profile
BGM profile
GPU memory policy
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

## 15. Narration Tool Stack

### Primary TTS

`CosyVoice 3`

Reasons for current selection:

```text
Korean is part of the model's multilingual training/use scope, not only UI-level support
strong multilingual / cross-lingual capability
zero-shot voice cloning
style / emotion / speed control potential
large and active ecosystem
suitable for future multi-channel voice expansion
local deployment possible
```

### Backup TTS Candidate

`GPT-SoVITS`

Role:

```text
future backup/fallback candidate
strong Windows/community ecosystem
voice cloning and channel-specific voice expansion
REST/API-friendly deployment
```

Do NOT wire the backup engine into v1 unless real CosyVoice failure data justifies the extra complexity.

### TTS Quality Control

`faster-whisper`

Recommended role:

```text
Narration generation ❌
Subtitle source ❌
TTS QC / pronunciation verification ✅
```

Potential checks:

```text
proper noun mismatch
number/ticker misread
missing phrase
unexpected substitution
major transcription divergence
```

QC may remain optional during the first implementation pass, but the interface should be reserved.

---

## 16. Subtitle Tool Stack

Primary subtitle source must be the original Planner subtitle/narration text, not ASR re-transcription.

Core path:

```text
Original Subtitle Text
+
Actual TTS timing/duration
↓
SRT Engine
↓
FFmpeg subtitle render
```

Reason:

```text
The exact Korean text is already known before TTS.
ASR re-transcription can introduce unnecessary spelling, spacing, proper-noun, or number errors.
```

### Advanced Optional Alignment

`WhisperX`

Use only when word-level timing is required, such as:

```text
karaoke-style captions
word highlighting
TikTok-style progressive subtitles
precise word-level effects
```

Do not make WhisperX mandatory for the baseline pipeline.

---

## 17. BGM Tool Stack

### Default

`Local BGM Library`

Planner outputs only:

```text
mood
style
tempo
intensity
```

n8n selects an appropriate local BGM asset.

This remains the default because it minimizes:

```text
GPU load
latency
generation failures
unpredictable output
pipeline complexity
```

### Optional AI BGM

`ACE-Step 1.5`

Use only when:

```text
local library has no suitable match
custom BGM is intentionally requested
experimental generation mode is enabled
```

Do not run ACE-Step automatically for every video.

---

## 18. GPU / VRAM Memory Policy

### Core Rule

**Release GPU memory by Engine Stage, not by Scene.**

Incorrect baseline behavior:

```text
Scene 1 complete
→ unload models
→ Scene 2 reload
→ unload
→ Scene 3 reload
```

This is prohibited as the default because repeated model reload creates unnecessary latency.

### Visual Stage Policy

```text
Scene 1 FLUX → LTX
↓
Scene 2 FLUX → LTX
↓
Scene 3 FLUX → LTX
↓
...
↓
All visual scenes complete
↓
Visual Stage Complete
↓
GPU Memory Release
```

During the Visual Stage:

```text
ComfyUI server may remain running continuously
FLUX/LTX models may stay resident/cached when beneficial
Do not force full VRAM unload between scenes
```

After all visual scenes are complete:

```text
request ComfyUI model/VRAM release
verify sufficient GPU memory is available
then start Audio Stage
```

### Audio Stage Policy

```text
CosyVoice generation batch
↓
CosyVoice stage complete
↓
release its GPU memory if required
↓
faster-whisper QC batch (optional)
```

### BGM Stage Policy

```text
Local BGM Library
→ no GPU generation required
```

If ACE-Step is required:

```text
Audio/TTS GPU work complete
↓
GPU memory release
↓
ACE-Step load
↓
Generate BGM
↓
ACE-Step release
```

### Concurrency Rule

On RTX 3060 12GB, do not intentionally keep these large GPU workloads active concurrently:

```text
FLUX/LTX
CosyVoice
faster-whisper
ACE-Step
```

Pipeline should favor sequential Stage execution over simultaneous GPU model residency.

### Memory Optimization Priority

```text
1. Avoid model reload between scenes
2. Avoid simultaneous large-model VRAM competition
3. Release memory at Stage boundaries
4. Keep services/processes running where possible
5. Unload model weights, not entire applications, unless required by stability
```

---

## 19. Final FFmpeg Compose

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

## 20. Final AV Validator

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

## 21. Run Manifest

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
  "audio_engine": "CosyVoice3",
  "subtitle_mode": "ORIGINAL_TEXT",
  "bgm_mode": "LOCAL_LIBRARY",
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
tool-version tracking
```

---

## 22. Path Layout

Recommended Run layout:

```text
/files/runs/{run_id}/
├─ anchors/
│  └─ scene_XX_anchor.png
├─ scenes/
│  └─ scene_XX.mp4
├─ tts_audio/
├─ bgm/
├─ subtitle.srt
├─ final_narration.mp3
├─ final_output.mp4
├─ shorts_final.mp4
└─ manifest.json
```

---

## 23. v0.71 Reuse / Replacement Policy

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
SRT generation framework
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
TTS integration
TTS metadata restoration
Subtitle timing source
Video/Narration/Subtitle Ready metadata
Final file validation
```

### Replace

```text
AnimateDiff T2V Prompt Engine
AnimateDiff ComfyUI generation request
legacy primary TTS engine if CosyVoice passes validation
Whisper-as-primary-subtitle-generation logic
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
CosyVoice integration
faster-whisper TTS QC interface
WhisperX optional alignment interface
Local BGM Library selector
ACE-Step optional generation interface
Stage-based GPU Memory Release
Final AV Validator
Run Manifest
```

---

## 24. Implementation Order

Implement one block at a time and preserve rollback after each verified stage.

```text
1. Validate CosyVoice 3 Korean output on RTX 3060 12GB
2. Confirm subtitle source = Original Text + timing
3. Confirm Local BGM Library structure
4. Reserve ACE-Step optional interface only
5. Add Stage-based GPU Memory Policy to Config
6. Pipeline Config
7. Qwen Production Planner
8. Plan Validator
9. Production Runtime
10. Scene Splitter
11. Scene Analyzer
12. Prompt Engine
13. FLUX generation + Anchor Check
14. LTX I2V generation
15. Visual Validator + Retry
16. Scene Collection adaptation
17. Visual Stage GPU Memory Release
18. CosyVoice integration
19. faster-whisper QC optional integration
20. SRT Runtime adaptation
21. ACE-Step optional integration only if needed
22. Final FFmpeg
23. Final AV Validator
24. Run Manifest
25. Full end-to-end verification
```

Do not promote this spec to FINAL implementation status until the redesigned workflow completes verified end-to-end execution.

---

## 25. Design Principles

```text
Visual Stability > Story/Concept Continuity > Natural Motion > Prompt Following > Fine Detail
```

Operational principles:

```text
local-first
free-first
Korean-first for narration quality
minimal engine count
no unnecessary multi-service routing
no fixed scene count
controlled retry only
no unlimited regeneration
preserve proven v0.71 downstream assets
separate creative planning from generation mechanics
separate target Config from actual Runtime
Original Text is the subtitle source of truth
Local BGM Library is the default BGM source
AI BGM is optional only
GPU models run sequentially by Stage on RTX 3060 12GB
VRAM release occurs at Stage boundaries, not Scene boundaries
keep services alive when possible to avoid restart overhead
human final decision remains authoritative
```

---

## 26. Tool Lock Status

### Locked for baseline implementation

```text
Visual: FLUX Anchor → LTX I2V
Narration Primary: CosyVoice 3
Subtitle Source: Original Text + SRT Engine
BGM Default: Local BGM Library
Final Compose: FFmpeg
Final Validation: ffprobe
```

### Reserved / optional

```text
TTS Backup: GPT-SoVITS
TTS QC: faster-whisper
Advanced Subtitle Alignment: WhisperX
AI BGM: ACE-Step 1.5
Visual Fallback: Wan I2V
```

Optional tools must not increase baseline complexity unless actual failure data or a feature requirement justifies enabling them.

---

## 27. Design Lock Decision

The redesign architecture, local tool stack, and GPU memory policy are sufficiently validated for implementation.

Next phase is execution, not further architecture expansion.

Any design change discovered during implementation must be documented, justified by actual test evidence, and validated before being merged into the locked specification.
