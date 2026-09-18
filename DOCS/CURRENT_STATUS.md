# AI Video Factory — Current Status

_Last updated: 2026-09-18_

## Current Decision

The local language-model baseline remains **Qwen2.5 14B Instruct Q4_K_M** on Ollama.

**Qwen3.5 9B Q4_K_M was tested and rejected for the main pipeline** despite stronger public benchmark scores. In our actual Korean Shorts workflow it showed concept drift, unnatural expressions, multilingual token contamination, excessive thinking when enabled, and weaker content stability. It may remain installed only as an experiment model.

## Current Pipeline Direction

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
Scene Collection
  ↓
CosyVoice3 TTS
  ↓
Whisper / SRT
  ↓
FFmpeg
  ↓
Final MP4
```

## LLM Operating Principle

### Concept Grounding
Accuracy first. Normalize slang, abbreviations, and ambiguous concepts before script generation.

### Fact Pack
Not an academic fact-checker. Its main job is to block **fatal misinformation and strong unsupported causal claims** while allowing reasonable simplification for public informational Shorts.

### Qwen2.5 Script Writer
Give it more expressive freedom. Prioritize:
- natural Korean
- information density
- hook and flow
- public-friendly wording
- engaging sentence rhythm

Do not over-constrain style just to maximize academic precision.

### Light Validator
Only fail on serious problems:
- topic/concept misunderstanding
- clear factual error
- strong unsupported medical/scientific claim
- broken Korean or multilingual token contamination
- narration/scene mismatch
- new facts introduced during scene splitting

Minor simplification, casual wording, mild exaggeration, or non-academic phrasing can PASS if the meaning is reasonable and there is no fatal error.

## Model Test Result

### Qwen2.5 14B
- Better fit for this project than Qwen3.5 9B.
- More stable Korean concept retention.
- Still weak as an independent Fact Engine, so Fact Pack / Validator separation remains necessary.
- Keep as Main Local LLM.

### Qwen3.5 9B
- Installation and Ollama API verified.
- `/api/chat` with `think:false` and UTF-8 request verified.
- Faster and stronger at structural instruction following.
- Failed practical content tests due to concept drift, odd Korean expressions, invented causal explanations, and multilingual contamination.
- Not selected as Main Script Writer.

### Gemini / ChatGPT web blind tests
- Both produced stronger cloud-model output than local Qwen in natural Korean and content construction.
- Gemini showed especially good Shorts/content expression.
- ChatGPT showed strong instruction following and balanced reasoning.
- **Paid API automation is not adopted at this stage.** Web results are used only as quality benchmarks during development.

## Stable Media Stack

- Image: FLUX anchor image
- Video: LTX 0.9.6 I2V
- Baseline resolution: 416×736
- Frames: 121 per scene
- Output: 20 fps
- LTX conditioning: 30 fps
- Scene duration: ~6.05 s
- 40 s target: 7 scenes
- TTS: CosyVoice3 local API on port 5051
- Subtitle: faster-whisper
- Final assembly: FFmpeg

## Development Rule

Work one node at a time. Validate each stage before connecting the next stage. Preserve known-good workflow versions before structural changes.

## Immediate Next Step

Optimize the **Qwen2.5-based Fact Pack → Script Writer → Light Validator → Scene Adapter** structure, with fewer style restrictions and fatal-error-only validation.
