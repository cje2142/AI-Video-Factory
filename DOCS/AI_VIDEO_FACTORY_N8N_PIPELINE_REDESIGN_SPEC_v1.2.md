# AI Video Factory
## n8n Pipeline Redesign Spec v1.2

**Status:** DESIGN + TTS VALIDATED / PRE-N8N-INTEGRATION BASELINE  
**Purpose:** Preserve the validated redesign state immediately before CosyVoice is integrated into n8n. This version keeps the v1.1 visual/runtime/VRAM design, replaces TTS assumptions with actual local validation results, and defines the next implementation boundary.

---

## 1. Baseline / Rollback Policy

Preserve unchanged:

- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/` — verified end-to-end rollback baseline
- `DOCS/AI_VIDEO_FACTORY_N8N_PIPELINE_REDESIGN_SPEC_v1.0.md`
- `DOCS/AI_VIDEO_FACTORY_N8N_PIPELINE_REDESIGN_SPEC_v1.1.md`

v1.2 is a new checkpoint. Do not overwrite v0.71, v1.0, or v1.1.

---

## 2. Current Validated Architecture

```text
Input / Config
↓
Qwen Production Planner
↓
Plan Validator
↓
Production Runtime
↓
Scene Splitter
↓
Scene Analyzer
↓
Prompt Engine
↓
FLUX Anchor
↓
Anchor Check
↓
LTX I2V
↓
Visual Validator
↓
Retry / Fallback Gate
↓
Scene Collection / Merge
↓
GPU Memory Release
↓
CosyVoice Narration
↓
Subtitle Timing / SRT
↓
Optional BGM
↓
FFmpeg Final Compose
↓
Final AV Validator
↓
Run Manifest
```

Main visual path remains:

```text
FLUX Anchor → LTX I2V
```

Wan I2V remains a reserved fallback interface only.

---

## 3. Runtime Rules Kept from v1.1

### Config vs Runtime

```text
Pipeline Config = desired target
Production Runtime = actual production values
```

Runtime owns at minimum:

```text
actual_scene_count
expected_scene_count
actual_total_duration
scene_durations
run_id
run_name
pipeline_version
```

No downstream fixed assumptions such as `10 scenes / 40 sec` are allowed.

### Scene Analyzer

Allowed Scene Types:

```text
DRONE_NATURE
CHARACTER_3D
SCREEN_UI
PRODUCT_OBJECT
REAL_HUMAN
```

Core rule:

```text
Risk ↑
→ Motion ↓
→ Shot simpler
→ Prompt simpler
```

### Prompt Engine

Planner handles meaning. Prompt Engine handles generation instructions.

```text
Scene Plan
+ Scene Type
+ Risk
+ Motion Level
↓
FLUX Prompt
FLUX Negative Prompt
LTX Prompt
LTX Negative Prompt
```

FLUX role:

```text
subject / composition / pose / environment / lighting / geometry
```

LTX role:

```text
preserve anchor / restrained motion / viewpoint change / continuity
```

---

## 4. Current LTX Baseline

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

Validated REAL_HUMAN direction:

```text
slow continuous motion
steady gaze
minimal eye movement
stable hands/body
avoid fast or jerky gestures
```

LTX T2V is not the main human-scene path because hand/body structure errors were less stable than I2V.

---

## 5. Visual Retry Policy

Validation states:

```text
PASS
MINOR
CRITICAL
```

Rules:

```text
Anchor failure
→ regenerate FLUX only

Valid Anchor + LTX CRITICAL
→ keep same Anchor
→ new LTX seed
→ retry once

Second CRITICAL
→ fallback_required = true
```

No unlimited regeneration.

---

## 6. GPU / VRAM Policy

**Release GPU memory by Engine Stage, not by Scene.**

Correct policy:

```text
All visual scenes
→ Visual Stage Complete
→ GPU Memory Release
→ Audio Stage
```

Do not unload FLUX/LTX after every scene because repeated reload creates unnecessary latency.

On RTX 3060 12GB, avoid intentional concurrent residency of large GPU workloads:

```text
FLUX / LTX
CosyVoice
faster-whisper
ACE-Step
```

Priority:

```text
1. Avoid scene-by-scene model reload
2. Avoid large-model VRAM competition
3. Release at Stage boundaries
4. Keep services running when practical
5. Unload weights rather than entire applications where possible
```

---

## 7. CosyVoice Validation Status

### Engine

Validated local model:

```text
FunAudioLLM/Fun-CosyVoice3-0.5B-2512
```

Local model directory:

```text
pretrained_models/Fun-CosyVoice3-0.5B
```

Current result:

```text
Korean zero-shot synthesis: PASS
Korean voice cloning: PASS
Local execution: PASS
WAV output: PASS
```

CosyVoice is therefore no longer only a candidate. It is the **validated primary TTS engine for the next n8n integration pass**.

GPT-SoVITS remains a backup candidate only and should not be added unless actual CosyVoice failure data justifies the extra complexity.

---

## 8. Voice Profile Strategy

Use a profile-based reference system rather than hard-coding one speaker.

Recommended logical structure:

```text
voice_profiles/
├─ <profile_id>/
│  ├─ reference.wav
│  ├─ sample_output.wav
│  └─ reference.txt
```

Each `reference.txt` should contain at minimum:

```text
PROFILE_ID
PROMPT_TEXT
REFERENCE_FILE
DESCRIPTION
SOURCE
SOURCE_CLIP
LICENSE
COMMERCIAL_USE
NOTES
```

### Validated public-reference profiles

#### `cv_female_bright_calm`

Observed characteristics:

```text
young Korean female
bright but calm
natural Korean pronunciation
slight synthetic character remains
usable for Shorts narration
```

Reference source:

```text
Mozilla Common Voice Korean
CC0-1.0
source clip: common_voice_ko_36880067.mp3
```

#### `cv_male_young_natural`

Observed characteristics:

```text
young Korean male
everyday / general-person feel
natural pronunciation
very low synthetic feel
strong practical usability
```

Reference source:

```text
Mozilla Common Voice Korean
CC0-1.0
source clip: common_voice_ko_36880087.mp3
```

### Private / user-owned references

User-owned voice references may be kept locally as separate profiles, but private voice assets and personally identifying profile metadata should **not** be committed to the public repository.

---

## 9. Common Voice Extraction Method

Working approach:

```text
Hugging Face dataset metadata
→ datasets streaming
→ decode(False)
→ gender filter
→ raw MP3 bytes
→ ffmpeg conversion to 24 kHz mono WAV
→ CosyVoice zero-shot test
```

Reason for `decode(False)`:

```text
Avoid torchcodec dependency and Windows DLL / PyTorch ABI conflicts.
```

Do not modify the working CosyVoice PyTorch environment merely to support dataset audio decoding.

Validated conversion target:

```text
24,000 Hz
mono
PCM WAV
```

---

## 10. TTS Prompt Rule

CosyVoice zero-shot prompt text must match the reference audio transcript.

Pattern:

```text
You are a helpful assistant.<|endofprompt|><exact reference transcript>
```

Keep:

```text
prompt_wav = selected profile reference.wav
prompt_text = exact transcript belonging to that reference
```

Do not reuse a transcript from another voice profile.

Longer, natural narration sentences produced slightly less synthetic output than very short test sentences in current tests.

---

## 11. TTS Quality Control

Initial production path:

```text
Planner narration
→ CosyVoice
→ WAV
```

Optional QC interface remains reserved for `faster-whisper`.

Recommended QC role only:

```text
proper noun mismatch
number / ticker misread
missing phrase
unexpected substitution
major transcription divergence
```

Do not use ASR as the authoritative subtitle source when the original text is already known.

---

## 12. Subtitle Policy

Primary source:

```text
Original Planner narration/subtitle text
+
actual generated audio timing
↓
SRT Engine
```

WhisperX remains optional for future word-level effects only.

---

## 13. BGM Policy

Default:

```text
Local BGM Library
```

Optional only when explicitly required:

```text
ACE-Step 1.5
```

Do not load AI BGM generation for every run.

---

## 14. Next Implementation Boundary

The redesign is now sufficiently validated to move from research/testing to integration.

### Next task 1 — Shared CosyVoice runner

Create a reusable local entry point:

```text
cosyvoice_generate.py
```

Input concept:

```text
text
voice_profile
output_path
```

Output:

```text
WAV file
status
profile_id
duration / metadata as needed
```

Requirements:

```text
no hard-coded single voice
select profile by ID
load exact reference.wav + prompt text
clear error output
safe output naming
n8n-friendly execution
```

### Next task 2 — n8n TTS replacement

Replace the v0.71 TTS execution block with the shared CosyVoice runner while preserving downstream audio/subtitle/FFmpeg logic where possible.

### Next task 3 — Visual integration

After TTS automation is stable:

```text
replace old DreamShaper / AnimateDiff visual block
with
FLUX Anchor → LTX I2V
```

Reuse proven History / Download / Save / Merge infrastructure.

---

## 15. Implementation Guardrails

```text
1. Do not modify v0.71 rollback baseline.
2. Do not overwrite v1.0 or v1.1 specs.
3. Do not add GPT-SoVITS until needed by evidence.
4. Do not add torchcodec to the CosyVoice environment for Common Voice extraction.
5. Do not unload visual models between every scene.
6. Do not hard-code one voice profile into n8n.
7. Do not commit private voice recordings to the public repository.
8. Prefer deterministic Code nodes over extra LLM calls where rules are sufficient.
9. Preserve Runtime-driven dynamic scene count/duration.
10. Validate one stage before adding the next dependency.
```

---

## 16. v1.2 Checkpoint Summary

```text
v0.71 end-to-end automation baseline: PRESERVED
FLUX → LTX I2V visual design: VALIDATED BASELINE
Dynamic Runtime design: LOCKED
Stage-based VRAM policy: LOCKED
CosyVoice3 Korean local TTS: VALIDATED
Female Common Voice profile: VALIDATED
Male Common Voice profile: VALIDATED
Common Voice extraction without torchcodec: VALIDATED
n8n CosyVoice integration: NOT YET IMPLEMENTED
FLUX/LTX n8n replacement: NOT YET IMPLEMENTED
```

**Checkpoint state:** `DESIGN + TTS VALIDATED / PRE-N8N-INTEGRATION BASELINE`
