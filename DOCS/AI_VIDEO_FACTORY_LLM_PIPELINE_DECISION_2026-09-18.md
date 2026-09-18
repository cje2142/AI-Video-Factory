# AI Video Factory — LLM Pipeline Decision Backup

_Date: 2026-09-18_

## Purpose

This document records the major decisions from the current redesign session so the next chat or development session can resume without repeating model-selection tests.

## 1. Main Local LLM Decision

Selected:
- **Qwen2.5 14B Instruct Q4_K_M**
- Runtime: Ollama
- Role: main local Korean Script Writer / structured content model

Rejected for main pipeline:
- **Qwen3.5 9B Q4_K_M**

Qwen3.5 was installed and tested because public benchmarks were significantly stronger than Qwen2.5 14B, but project-specific testing showed that benchmark superiority did not translate into better Korean informational Shorts output.

## 2. Qwen3.5 Test Findings

Verified technically:
- Ollama model installation completed.
- `/api/chat` works.
- `think:false` works when sent as an API parameter.
- UTF-8 PowerShell request works when JSON is encoded to UTF-8 bytes.

Practical failures observed:
- Misread Korean slang/context: `우다다` was repeatedly interpreted as cat crying/howling instead of fast running.
- Concept drift returned even after initial normalization in some tests.
- Unsupported causal claims were generated confidently.
- Korean phrasing sometimes became unnatural or strange.
- Chinese/foreign-language contamination appeared in output.
- Thinking-enabled runs became excessively long without reliably correcting the initial concept error.
- Thinking-off improved speed but not enough to solve content-quality instability.

Decision:
- Keep Qwen3.5 only as an experimental model.
- Do not replace Qwen2.5 in the main production pipeline.

## 3. Qwen2.5 Test Findings

Strengths:
- Better concept retention for Korean content.
- More stable fit with the existing n8n workflow.
- Adequate natural-language quality when style constraints are not excessive.
- Predictable behavior for JSON and structured output.

Weaknesses:
- Can invent plausible but weakly supported factual explanations.
- Overly strict fact/professional-language constraints reduce expression quality.
- Should not be trusted as a standalone fact checker.

Decision:
- Keep Qwen2.5 14B as the project baseline.
- Solve accuracy problems structurally instead of swapping to Qwen3.5.

## 4. Validation Philosophy Changed

Previous direction:
- heavy factual/professional constraints
- many explicit prohibitions
- strong self-check requirements

Observed problem:
- Korean became stiff
- information density dropped
- creative expression declined
- model spent effort satisfying rules instead of writing a good Shorts script

New direction:
- Allow public-friendly simplification.
- Allow expressive wording if meaning is reasonable.
- Do not require academic-level precision for every sentence.
- Only block fatal errors.

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

### FAIL
- concept/topic misunderstanding
- clear factual misinformation
- dangerous or strong unsupported medical/scientific claim
- sentence corruption
- Chinese/foreign token contamination
- narration/scene inconsistency
- new factual claims added during scene splitting

## 5. Target Architecture

```
Pipeline Config
  ↓
Prepare Run Folder
  ↓
Concept Grounding
  ↓
Fact Pack
  ↓
Qwen2.5 Script Writer
  ↓
Light Validator
  ↓
Scene Adapter
  ↓
Prompt Engine
  ↓
FLUX Anchor Image
  ↓
LTX I2V
  ↓
Visual Validation / Scene Collection
  ↓
CosyVoice3 TTS
  ↓
Whisper / SRT
  ↓
FFmpeg
  ↓
Final MP4
```

## 6. Role Boundaries

### Concept Grounding
- Normalize slang / ambiguous topic.
- Example: `우다다` → `고양이가 갑자기 집 안을 매우 빠르게 뛰어다니는 행동(zoomies)`.
- Accuracy over style.

### Fact Pack
- Provide a small set of usable facts and boundaries.
- Do not aim for academic completeness.
- Main purpose: prevent fatal misinformation and overconfident unsupported claims.

### Qwen2.5 Script Writer
- Use Fact Pack as source material.
- Freedom to paraphrase naturally.
- Prioritize hook, information density, natural Korean, rhythm, and audience accessibility.
- Do not invent strong new facts or medical/scientific certainty.

### Light Validator
- Fatal-error gate only.
- Must not rewrite acceptable creative expressions into stiff academic language.

### Scene Adapter
- Split completed narration into coherent scene units.
- Do not introduce new facts.
- Keep narration meaning intact.
- Produce subtitle and simple visual intent.

### Prompt Engine
- Own technical FLUX/LTX prompt transformation.
- Content Planner should not carry technical camera/model constraints.

## 7. Cloud Model Benchmark Decision

Blind tests were done in logged-out web sessions to reduce personalization influence.

Observed:
- Gemini: strongest content/Shorts expression among tested outputs.
- ChatGPT: strongest balance of instruction following, reasoning, and factual restraint.
- Qwen local: useful but clearly weaker as an all-in-one researcher + writer + validator.

However:
- paid API automation is not allowed at the current project stage.
- Gemini/ChatGPT are therefore retained only as manual benchmark references.

## 8. Stable Media Pipeline Reference

- FLUX Anchor → LTX 0.9.6 I2V
- 416×736
- 121 frames per scene
- 20 fps output
- 30 fps conditioning
- ~6.05 s per scene
- 7 scenes for ~40 s target
- CosyVoice3 local TTS API: port 5051
- faster-whisper subtitles
- FFmpeg final assembly

## 9. Next Development Task

Design and test the production prompts/nodes for:
1. Fact Pack
2. Qwen2.5 Script Writer
3. Light Validator
4. Scene Adapter

Main design requirement:
**Do not sacrifice expression quality for excessive factual policing. Only fatal content errors should stop the pipeline.**
