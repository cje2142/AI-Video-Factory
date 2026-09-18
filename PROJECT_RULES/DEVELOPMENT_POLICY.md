# AI Video Factory Development Policy v1.2

## 0. Authority and Scope
- This file defines the project-wide development and operating policy.
- `DOCS/CURRENT_STATUS.md` defines the latest verified project state and currently selected stack.
- Dated decision documents under `DOCS/` preserve the evidence and reasoning behind major decisions.
- When older specifications conflict with the latest verified state, preserve the older file as historical evidence and follow the latest verified state unless a rollback is explicitly requested.
- Do not silently overwrite a known-good baseline or historical specification.

## 1. Roles
- ChatGPT: project planning, architecture, implementation guidance, step-by-step progress control, validation, and documentation.
- External AI models/services: optional benchmark or specialist tools only when they add verified value. Do not add them to the production path without evidence.
- User: executes local tests, confirms observed results, and retains final approval over project changes.
- Human-observed test results take priority over benchmark claims when selecting production components.

## 2. Development Principles
- Proceed one verified stage or node at a time.
- Do not modify a known-good workflow unnecessarily.
- Test each new stage separately before integration.
- Do not change multiple major components at once unless required for a single inseparable change.
- For code or node logic changes, prefer complete revised code/configuration over fragile partial patches.
- Preserve a rollback point before structural changes.
- Prefer deterministic Code nodes over extra LLM calls when rules are sufficient.
- Keep the engine/service count minimal. Local-first and free-first remain the default unless measured results justify added complexity.
- Separate content planning from generation mechanics.

## 3. Current Production Architecture
Current target route:

```text
Pipeline Config
→ Prepare Run Folder
→ Concept Grounding
→ Fact Pack
→ Qwen2.5 14B Script Writer
→ Light Validator
→ Scene Adapter
→ Prompt Engine
→ FLUX Anchor Image
→ LTX 0.9.6 I2V
→ Visual Validation / Scene Collection
→ GPU Memory Release
→ CosyVoice3 TTS
→ Subtitle Timing / faster-whisper QC / SRT
→ Optional Local BGM
→ FFmpeg Final Compose
→ Final AV Validation
→ Final MP4 / Run Manifest
```

- Qwen2.5 14B Instruct Q4_K_M on Ollama is the current main local language model.
- Qwen3.5 9B is experimental only and is not part of the main production path.
- FLUX Anchor → LTX I2V is the current main visual path.
- Wan I2V remains a reserved fallback interface unless later evidence promotes it.
- CosyVoice3 is the current primary local TTS engine.
- FFmpeg remains the deterministic final compositor.

## 4. Config and Runtime Rules
- `Pipeline Config` stores desired targets.
- `Production Runtime` stores actual production values.
- Scene count and total duration must be runtime-driven; no fixed 5-scene, 7-scene, 10-scene, or fixed-duration assumption may be embedded in downstream logic.
- Any example such as “7 scenes ≈ 40 s” is a current profile reference, not a hard rule.
- Runtime should own at minimum:
  - actual_scene_count
  - expected_scene_count
  - actual_total_duration
  - scene_durations
  - run_id
  - run_name
  - pipeline_version
- Downstream completion checks must use expected Runtime values, not hard-coded counts.

## 5. LLM Content Boundaries
### Concept Grounding
- Normalize slang, abbreviations, ambiguous concepts, and likely interpretation errors before script generation.
- Accuracy takes priority over style at this stage.

### Fact Pack
- Provide a compact set of usable facts, limits, and uncertainty boundaries.
- Do not aim for academic completeness.
- Main purpose: prevent fatal misinformation and strong unsupported causal claims.

### Qwen2.5 Script Writer
- Use the Fact Pack as source material.
- Prioritize natural Korean, information density, hook, flow, rhythm, and public accessibility.
- Allow reasonable paraphrasing, simplification, analogy, and expressive wording.
- Do not invent strong new facts or unsupported medical/scientific certainty.

### Light Validator
- Operate as a fatal-error gate, not an academic rewrite engine.
- Do not rewrite acceptable creative language into stiff professional or academic prose.

### Scene Adapter
- Split the completed narration into coherent scene units.
- Preserve meaning and factual boundaries.
- Do not introduce new factual claims.
- Produce subtitle text and simple visual intent only.

### Prompt Engine
- Own FLUX/LTX generation instructions and technical prompt transformation.
- Content-planning nodes must not carry model-specific camera, sampler, seed, or generation mechanics unless explicitly required.

## 6. Content Validation Policy
Use PASS / WARN / FAIL.

### PASS
- natural simplification
- casual Korean
- mild exaggeration
- accessible analogies
- non-academic wording
- slightly uncertain but reasonable descriptions

### WARN
- somewhat loose causal wording
- weakly supported general explanation
- minor repetition
- slightly aggressive hook

WARN does not automatically stop production.

### FAIL
- topic or concept misunderstanding
- clear factual misinformation
- dangerous or strong unsupported medical/scientific claim
- broken/corrupted Korean
- unintended Chinese or foreign-token contamination
- narration/scene inconsistency
- new factual claims introduced during scene splitting

FAIL stops or returns the affected stage for correction.

## 7. Media Validation
- Verify actual file creation, not only node success.
- Verify n8n binary fields and paths when binary transfer is used.
- Verify ComfyUI History before treating asynchronous generation as complete.
- Wait/poll until all Runtime-expected scene outputs are confirmed.
- Anchor failure: regenerate FLUX only.
- Valid Anchor + critical LTX failure: preserve Anchor, retry LTX with a new seed within the configured retry limit.
- No unlimited regeneration.
- Verify final file existence, video/audio streams, duration range, scene completeness, subtitle presence, decodability, file size, and output path before declaring final success.

## 8. Current Stable Media Profile
Reference profile, not a permanent hard-coded rule:
- Resolution: 416×736
- LTX: 121 frames per scene
- Output: 20 fps
- LTX conditioning: 30 fps
- Approximate scene duration: 6.05 s
- RIFE: OFF unless later testing changes the baseline
- TTS: CosyVoice3 local API
- Subtitle/QC: faster-whisper as needed
- Final assembly: FFmpeg

Any change to this profile must be supported by actual comparative test evidence.

## 9. GPU / VRAM Policy
- Release GPU memory by engine stage, not after every scene.
- Preferred order:
  1. complete visual generation
  2. release visual-model GPU memory
  3. run audio/subtitle stages
- Avoid intentional concurrent residency of large GPU workloads on the RTX 3060 12GB.
- Avoid scene-by-scene FLUX/LTX reloads unless required by a proven stability issue.
- Prefer unloading model weights over restarting entire applications when practical.

## 10. Backup and Versioning
- GitHub is the official project backup.
- Preserve n8n and ComfyUI backups under their dedicated workflow/backup structure.
- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/` remains a protected verified rollback baseline.
- Do not overwrite known-good baselines or locked historical specs.
- Create a new version/checkpoint for meaningful structural changes.
- Before a major redesign, preserve the current working workflow and write clear restore instructions.
- Public repository backups must not contain private voice recordings or personally identifying private assets.

## 11. Recovery
If a change fails validation:
1. stop expanding the change,
2. return to the last verified stage or version,
3. identify one failure cause at a time,
4. fix and retest that stage,
5. reconnect downstream stages only after validation passes.

Do not compensate for one unstable component by adding multiple new engines or services at once.

## 12. Unified Workspace Root
- Default Windows project workspace root: `D:\chatgpr`.
- New project-owned files, scripts, temporary working files, test outputs, utilities, and handoff assets should be created under this root whenever technically practical.
- Use a dedicated project subfolder rather than scattering files across Desktop, Downloads, or temporary directories.
- Existing applications, model repositories, installed runtimes, and validated environments may remain in their current paths when moving them would add risk.
- Before introducing a new top-level path outside `D:\chatgpr`, verify that the tool or runtime actually requires it.

## 13. Change-Control Rule
A component may be promoted into the main production path only after:
1. isolated test success,
2. comparison against the current baseline where relevant,
3. no regression in downstream integration,
4. observed benefit that justifies added complexity,
5. backup of the previous known-good state.

Benchmark superiority alone is not sufficient for promotion.

## 14. Immediate Development Boundary
The next optimization scope is:

```text
Fact Pack
→ Qwen2.5 Script Writer
→ Light Validator
→ Scene Adapter
```

Complete and validate this content-generation block before expanding architecture further.
