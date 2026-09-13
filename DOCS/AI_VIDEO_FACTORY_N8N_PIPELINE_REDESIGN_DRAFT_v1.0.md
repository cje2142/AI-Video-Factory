# AI Video Factory
## n8n Pipeline Redesign Draft v1.0

**Status:** Draft / Backup Baseline  
**Purpose:** Existing n8n workflow redesign baseline for the FLUX → LTX I2V production pipeline  
**Target Environment:** Local-first, RTX 3060 12GB, n8n + Ollama/Qwen + ComfyUI + FFmpeg  

---

## 1. Core Architecture

```text
User + ChatGPT
↓
Pipeline Config
↓
Ollama / Qwen Production Planner
↓
Plan Validator
↓
Scene Splitter
↓
Scene Analyzer
  ├─ Scene Type
  ├─ Risk Evaluation
  └─ Motion Level
↓
Prompt Engine
↓
FLUX Anchor Image
↓
Anchor Check
↓
LTX I2V
↓
Visual Validator
↓
Retry / Fallback
↓
Scene Collection
↓
TTS / Whisper / SRT / BGM
↓
FFmpeg Final Compose
↓
Final AV Validator
↓
Run Manifest
↓
Final MP4
```

---

## 2. Design Principles

- Do not discard the existing working n8n workflow.
- Classify existing nodes as `KEEP / MODIFY / DELETE / NEW` before implementation.
- Reuse verified assets wherever possible.
- Keep the architecture simple and local-first.
- Prefer free/local components before external paid services.
- Avoid duplicated engines and unnecessary routing layers.
- Use Run ID based Scene and Output management.
- Manage workflow versions by workflow name.
- Preserve rollback points before major changes.

Verified assets to reuse:

```text
Pipeline Config
Scene Split
ComfyUI API
prompt_id / History Polling
Scene file collection
TTS
Whisper
SRT
FFmpeg
Run ID / Output management
```

---

## 3. Input Rules

Required:

```text
concept
```

Optional:

```text
target_duration
audience
tone
voice_style
bgm_style
quality_mode
```

Do not hard-code the workflow to one content domain. Existing investment-content hard-coding should be removed from the main pipeline.

---

## 4. Pipeline Config

Separate creative and technical settings.

```text
Pipeline Config
├─ Creative Config
│  ├─ concept
│  ├─ target_duration
│  ├─ audience
│  ├─ tone
│  ├─ voice_style
│  ├─ bgm_style
│  └─ quality_mode
│
└─ Technical Config
   ├─ scene_duration_target
   ├─ generation_engine
   ├─ fallback_engine
   ├─ resolution
   ├─ fps
   ├─ retry_limit
   ├─ validation_enabled
   ├─ run_id
   └─ output_path
```

Technical constants should be centralized here rather than duplicated across nodes.

---

## 5. Production Planner

Ollama/Qwen handles content planning only.

```text
Concept
↓
Core Intent
↓
Story Plan
↓
Script
↓
Scene Plan
↓
BGM Direction
```

Planner output per Scene should include at minimum:

```text
scene_id
purpose
narration
subject
action
environment
framing
camera
visual_continuity
duration
```

Role rule:

```text
Planner = What to show
Scene Analyzer / Prompt Engine = How to generate it safely
```

---

## 6. Plan Validator

Validate the plan before GPU generation.

Prefer deterministic checks where possible:

```text
scene_count
scene_id duplication
missing narration
missing required fields
duration total
scene repetition
story discontinuity
concept drift
```

Do not use AI judgment for checks that can be done deterministically.

---

## 7. Dynamic Scene Split

Remove fixed 5-Scene / 10-Scene assumptions.

- Total video target: about 40–90 seconds.
- Scene target duration: about 4–6 seconds.
- Scene count is determined dynamically from content and duration.
- Long-form generation in a single video-engine call is not used.
- Generate by Scene and compose later.

`expected_scene_count` must replace any fixed scene-count logic.

---

## 8. Scene Analyzer

Scene Analyzer is separated from the Production Planner.

For every Scene determine:

```text
Scene Type
Risk
Motion Level
```

Current Scene Types:

```text
DRONE_NATURE
CHARACTER_3D
SCREEN_UI
PRODUCT_OBJECT
REAL_HUMAN
```

Core rule:

```text
Scene Type
→ Risk
→ Motion
→ Type-specific Prompt Rule
```

Risk principle:

```text
Risk ↑
→ Motion ↓
→ Shot simpler
→ Prompt simpler
```

Scene Analyzer must not redesign the story. It only converts the Scene Plan into a safe generation strategy.

---

## 9. Prompt Engine

Prompt Engine receives the structured Scene Plan plus Scene Analyzer output.

Output:

```text
FLUX Prompt
FLUX Negative Prompt
LTX Motion Prompt
LTX Negative Prompt
```

Prompt Engine follows the validated Prompt & Scene Rule v1.0.

FLUX role:

```text
Subject
Composition
Pose
Environment
Lighting
Geometry
```

LTX role:

```text
Preserve Anchor structure
Add Motion
Add Viewpoint / Perspective change
Maintain Continuity
```

The Prompt Engine must not absorb Planner responsibilities.

---

## 10. Image / Video Generation

Default production path:

```text
FLUX Anchor Image
↓
LTX I2V
```

FLUX fixes the starting state first:

```text
person / character
hands
composition
expression
background
geometry
```

LTX adds restrained motion while preserving the Anchor.

For human Main Scenes, LTX T2V is not the default path.

---

## 11. Anchor Check

Before LTX generation, perform a lightweight Anchor Check.

Initial deterministic checks:

```text
file exists
image decodes
expected resolution
minimum file size
valid output path
```

Do not initially add a heavy AI Vision validator unless actual failure data proves it is needed.

---

## 12. Visual Validation

Visual Validator output:

```text
PASS
MINOR
CRITICAL
```

Rules:

```text
PASS
→ accept

MINOR
→ accept if usable

CRITICAL
→ regenerate once with new seed
```

If the regenerated result is still CRITICAL:

```text
→ consider Fallback Engine
```

No unlimited regeneration.

Critical examples include severe face/hand/body deformation, identity change, major scene mismatch, strong flicker, broken frames, major geometry collapse, or major object morphing.

---

## 13. Retry / Fallback Strategy

Default path:

```text
FLUX Anchor
→ LTX I2V
→ Visual Validator
```

On CRITICAL:

```text
same FLUX Anchor
+ new LTX seed
→ LTX regenerate once
```

Do not regenerate FLUX automatically when the failure is clearly in LTX motion.

If CRITICAL again:

```text
Fallback candidate = Wan I2V
```

Do not build a complex Scene-by-Scene multi-engine Router at this stage.

---

## 14. RIFE Rule

RIFE is optional post-processing, not a mandatory part of the generation engine.

```text
Generation
↓
Optional RIFE
↓
Validation / downstream processing
```

If current LTX output fps and motion quality are sufficient, RIFE remains OFF.

---

## 15. Scene Collection

Reuse current prompt_id / History Polling / output collection logic.

Replace fixed-count completion checks with:

```text
expected_scene_count == completed_scene_count
```

Every collected Scene should be tied to:

```text
run_id
scene_id
prompt_id
output_file
validation_status
retry_count
```

---

## 16. TTS / Whisper / Subtitle

Reuse the verified audio/subtitle path.

```text
Scene Narrations
↓
Continuous Narration Text
↓
TTS
↓
Narration Audio
↓
Whisper
↓
Timestamp
↓
SRT
```

Continuous Narration is preferred over isolated Scene-by-Scene voice synthesis when it improves natural continuity.

---

## 17. BGM

Planner decides only BGM direction, not complex generation logic.

Example output:

```json
{
  "use": true,
  "style": "technology",
  "energy": "medium"
}
```

Initial implementation should prefer a small safe local BGM library.

BGM remains optional.

---

## 18. FFmpeg Final Compose

FFmpeg is the deterministic assembly engine.

Inputs:

```text
Scene Videos
Narration Audio
SRT
BGM (optional)
```

Output:

```text
Final MP4
```

Do not put creative decision logic inside the FFmpeg stage.

---

## 19. Final AV Validation

Prefer deterministic validation over generative judgment.

Check:

```text
final file exists
video stream exists
audio stream exists
expected duration range
audio/video duration difference
subtitle file exists
scene count complete
file size valid
file decodes
output path valid
```

Use ffprobe or equivalent deterministic checks where practical.

Final success state:

```text
FINAL_PASS
```

---

## 20. Run Manifest

Create one JSON manifest per run.

Recommended structure:

```json
{
  "run_id": "",
  "concept": "",
  "scene_count": 0,
  "scenes": [
    {
      "scene_id": 1,
      "flux_seed": 0,
      "ltx_seed": 0,
      "retry_count": 0,
      "validation": "PASS",
      "output": ""
    }
  ],
  "tts": "",
  "subtitle": "",
  "bgm": "",
  "final_validation": "",
  "final_file": ""
}
```

Purpose:

- track engine failure rate
- track regeneration rate
- track fallback usage
- compare prompt/risk patterns
- preserve reproducibility and debugging history

---

## 21. Recommended Logical Blocks

Keep the mental model to seven logical Blocks even if each Block contains several n8n nodes.

```text
BLOCK A — Input / Config
BLOCK B — Planning / Validation / Scene Split
BLOCK C — Scene Analyzer / Prompt Engine
BLOCK D — FLUX / LTX / Visual Validation / Retry
BLOCK E — Scene Collection
BLOCK F — TTS / Whisper / SRT / BGM
BLOCK G — FFmpeg / Final Validation / Output
```

---

## 22. Existing Workflow Redesign Classification

The existing workflow should be reviewed node-by-node using:

```text
KEEP
MODIFY
DELETE
NEW
```

Initial expected direction:

| Function | Initial Classification |
|---|---|
| Trigger / Input | MODIFY |
| Run ID | KEEP |
| Pipeline Config | MODIFY |
| Existing Script Generator | MODIFY |
| Existing Scene Parser | MODIFY |
| Scene Split | KEEP / MODIFY |
| Investment-domain hard-coding | DELETE |
| Existing Prompt Engine | MODIFY |
| FLUX API | KEEP / MODIFY |
| AnimateDiff main generation | DELETE from Main / retain as legacy test asset |
| LTX I2V | NEW |
| ComfyUI HTTP Request | KEEP |
| prompt_id management | KEEP |
| History Polling | KEEP |
| Fixed scene-count logic | MODIFY |
| Scene Analyzer | NEW |
| Visual Validator | NEW |
| Retry Controller | NEW |
| Wan Fallback | LATER / NEW |
| Scene Collection | KEEP / MODIFY |
| TTS | KEEP |
| Whisper | KEEP |
| SRT | KEEP |
| BGM | KEEP / MODIFY |
| FFmpeg | KEEP |
| Final Validator | NEW |
| Run Manifest | NEW |

Final classification must be based on the actual current workflow JSON, not this preliminary table alone.

---

## 23. Quality Priority

```text
1. Visual Stability
2. Story / Concept Continuity
3. Natural Motion
4. Prompt Following
5. Fine Detail / Sharpness
```

Maximum sharpness is secondary to avoiding visually disturbing structural errors.

---

## 24. Scope Exclusions

Not included in this redesign stage:

```text
complex multi-engine routing
multiple engines generating simultaneously
unlimited regeneration
content-specific separate workflows
automatic YouTube / TikTok upload
excessive Planner node splitting
external services without clear necessity
```

---

## 25. Final Automation Goal

The user provides only the subject and optional conditions.

The pipeline automatically performs:

```text
Planning
→ Scene Design
→ Scene Analysis
→ Prompt Generation
→ FLUX Anchor
→ LTX Video
→ Quality Validation
→ Retry / Fallback if needed
→ Scene Collection
→ Voice
→ Subtitle
→ BGM
→ Final Composition
→ Final Validation
→ Final MP4
```

This Draft is the redesign baseline. Implementation changes discovered during actual node-by-node review should be documented and validated before this document is promoted to FINAL v1.0.
