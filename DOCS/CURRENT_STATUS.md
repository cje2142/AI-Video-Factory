# AI Video Factory — Current Status

_Last updated: 2026-09-19_

## Current Verified Checkpoint

Current promoted n8n workflow:

- **AI Shorts Automation Pipeline v2.32-Final**
- 68 nodes
- 66 connection source entries
- operational export has `pinData = {}`
- Expanded content branch runtime-tested through safe STOP behavior

Detailed checkpoint:

- `DOCS/AI_VIDEO_FACTORY_AUTOMATION_CHECKPOINT_2026-09-19.md`

The protected rollback baseline remains:

- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

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

Evidence Expansion is allowed **once only**.

If the expanded script is still too short:

```text
needs_evidence_expansion = false
ready_for_scene_adapter = false
next_route = stop
→ Stop - Insufficient Verified Content
```

## Verified Expanded Node Versions

- `Fact Pack Input Expanded`: v1.7
- `Fact Pack Validator Expanded`: v1.6.2
- `Script Writer Input Expanded`: v1.2
- `Script Validator Expanded`: v1.6 with second expansion disabled

## Evidence / Fact Policy

Current production rule:

- Facts must come from supplied Evidence.
- Exact support quotes are required.
- Do not infer unsupported cause, purpose, motivation, onset, duration, frequency, or certainty.
- One-source propositions remain `MEDIUM + QUALIFY`.
- Script Writer paraphrases validated Facts only.
- Do not create filler to hit duration.
- If verified information is insufficient, stop safely.

## Latest Runtime Test

Concept:

- `고양이가 갑자기 우다다하는 이유`

Target:

- 40 s

Result:

- base script estimate: 15.9 s
- Evidence Expansion executed once
- successful isolated expansion sample: 73 search results / 37 sources / 1 new strong Tier-A source
- the new source was kitten-scoped and could not safely produce an additional general-cat Fact
- expanded Fact/Script integrity: PASS
- expanded duration preflight: FAIL
- final route: safe STOP

This behavior is considered **correct**.

## Current Main Blocker

The main content-branch blocker is now **external search reliability**, not Fact Pack validation.

Observed SearXNG upstream issues:

- Brave rate limiting
- DuckDuckGo CAPTCHA
- Wikidata timeout/rate-limit behavior
- zero-result expansion reruns despite a previously successful search

Next task: improve search resilience while preserving evidence rules.

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

## Known Deferred Cleanup

TTS metadata/runtime alignment is deferred:

- workflow metadata still contains CosyVoice3 references
- recent local service operation used an Edge-TTS-compatible container/image

Handle this later as an isolated verified change.

## Development Rule

Follow `PROJECT_RULES/DEVELOPMENT_POLICY.md`.

One verified node/stage at a time. Do not modify known-good components without a concrete runtime reason.

## Immediate Next Step

**SearXNG / Evidence Expansion search resilience**

Then:

1. rerun full content route without pinned data
2. validate a positive Expansion case that adds a new Fact
3. validate Expanded success → Scene Adapter
4. continue end-to-end media validation


## GitHub Backup Status

v2.32-Final workflow artifact backup is complete.

- Canonical: `WORKFLOWS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final.json`
- Archive: `BACKUPS/n8n/AI_Shorts_Automation_Pipeline_v2.32_Final_2026-09-19.json`
- Both files are 413,600 bytes and resolve to the same Git blob SHA:
  `78e9279113330fc9abdecb92ead4136d659fa0e1`

GitHub is now a verified restore source for this checkpoint.
