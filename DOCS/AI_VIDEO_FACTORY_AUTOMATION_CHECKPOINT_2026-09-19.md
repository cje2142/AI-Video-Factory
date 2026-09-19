# AI Video Factory — Automation Checkpoint

_Date: 2026-09-19_
_Status: v2.32-Final content/expansion branch validated_

## Purpose

This checkpoint records the verified n8n automation state, the evidence-grounded content design, the Evidence Expansion behavior, the latest runtime test results, and the next development direction.

## 1. Current Workflow Checkpoint

Current promoted workflow:

- **AI Shorts Automation Pipeline v2.32-Final**
- n8n nodes: **68**
- connection source entries: **66**
- `pinData = {}` in the exported operational workflow
- test-only pinned Evidence data removed before promotion
- JSON structure and Code-node syntax validated before promotion

The protected older rollback baseline remains:

- `WORKFLOWS/n8n/v0.71_FULL_AUTOMATION_BASELINE/`

Do not overwrite that baseline.

## 2. Current Content Architecture

Verified main content route:

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
→ Need Evidence Expansion?
```

Normal success route:

```text
Script Validator
→ Script Ready?
→ Scene Adapter
```

Short-script recovery route:

```text
Script Validator
→ Need Evidence Expansion? = true
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

Expanded route policy:

- **Evidence Expansion is allowed only once.**
- If the expanded script still fails duration preflight:
  - `needs_evidence_expansion = false`
  - `ready_for_scene_adapter = false`
  - `next_route = stop`
- False route ends at:
  - `Stop - Insufficient Verified Content`

The pipeline must never loop into repeated Evidence Expansion.

## 3. Expanded Branch Version Alignment

The duplicated Expanded branch had version drift. It was corrected and validated.

Current verified versions:

- `Fact Pack Input Expanded` → **v1.7**
- `Fact Pack Validator Expanded` → **v1.6.2**
- `Script Writer Input Expanded` → **v1.2**
- `Script Validator Expanded` → **v1.6**, with second expansion disabled

Ollama HTTP nodes remain shared-equivalent to the base branch.

No unrelated known-good node was intentionally modified during this checkpoint.

## 4. Fact/Evidence Safety Rules

The production content design is now evidence-first rather than free-form fact generation.

Key rules:

- Evidence snippets are the factual boundary.
- Fact Pack must extract, not invent.
- Exact contiguous `support_quotes` are required for every Fact.
- One source supporting a proposition:
  - `MEDIUM + QUALIFY`
- `HIGH + ALLOW` requires at least two independent Evidence sources supporting the same proposition.
- Do not infer:
  - cause
  - purpose
  - motivation
  - onset
  - frequency
  - duration beyond the quote
  - certainty stronger than the Evidence
- “burst of energy” is not automatically a cause.
- “seemingly out of nowhere” is not “there is no reason”.
- behavior itself and a change in behavior must remain distinct.
- Script Writer may paraphrase validated Facts only.
- The Script Writer must not invent filler merely to reach target duration.
- Insufficient verified content must stop the pipeline rather than manufacture content.

## 5. 2026-09-19 Runtime Validation — Cat Zoomies Test

Test concept:

```text
고양이가 갑자기 우다다하는 이유
```

Target duration:

- 40 s
- current runtime reference: 7 scenes

Base script result:

- estimated duration: **15.9 s**
- duration ratio: **0.398**
- content/fact integrity: PASS
- duration preflight: FAIL
- Evidence Expansion requested correctly

Successful Expansion search sample used for isolated validation:

- queries: 6
- raw results: 73
- raw sources: 37
- original sources: 6
- novel strong candidates: 1
- selected new strong source: 1
- Expansion status: WARN
- `ready_for_fact_pack = true`

New strong Evidence:

- VCA — `Understanding Kitten Zoomies`
- Tier A
- important scope limitation: **kitten**

Expanded Fact Pack result:

- Fact integrity: PASS
- no unsafe kitten → all-cats generalization
- no unsupported causal claim
- no new safe Fact could be added from the new Evidence

Expanded script result:

- still estimated at **15.9 s**
- all content validation checks PASS
- duration preflight FAIL

Final route was verified:

```text
Script Validator Expanded
→ status = FAIL
→ needs_evidence_expansion = false
→ ready_for_scene_adapter = false
→ next_route = stop
→ Expanded Script Ready? False
→ Stop - Insufficient Verified Content
→ stage = pipeline_stopped
```

This is the intended safety behavior.

## 6. Search Reliability Issue Found

The main remaining blocker in the content branch is not Fact Pack logic. It is external search reliability.

Observed during `SearXNG Search - Expansion`:

- Brave: `too many requests`
- DuckDuckGo: CAPTCHA
- Wikidata: timeout / rate-limit behavior
- a later rerun returned zero results for all six expansion queries

Interpretation:

- the Evidence Expansion logic works when search data exists
- external engine availability can cause false content insufficiency
- search resilience must be improved without weakening evidence rules

## 7. Next Search-Layer Direction

Next optimization scope:

1. improve SearXNG engine reliability and engine selection
2. add deterministic retry/backoff only where useful
3. preserve successful search/Evidence outputs so downstream retests do not require external search reruns
4. consider cached Evidence reuse for the same normalized topic/query set
5. distinguish:
   - true “no verified Evidence”
   - temporary search-provider failure
6. keep one-expansion maximum in production

Do **not** solve search instability by allowing the LLM to invent missing Facts.

## 8. Stable Media Direction

Current visual production baseline remains:

- FLUX anchor image
- LTX 0.9.6 I2V
- 416×736
- 121 frames per scene
- output 20 fps
- LTX conditioning 30 fps
- approximately 6.05 s generated duration per scene
- fixed-camera / simple-motion bias
- RIFE OFF unless later comparative evidence changes the baseline
- FFmpeg final assembly

The current workflow remains runtime-driven for scene count rather than relying on a permanently fixed 7-scene rule.

## 9. Known Runtime/Metadata Debt

Do not mix these fixes into the current content validation work unless necessary:

- `Pipeline Config` still contains older TTS metadata referring to CosyVoice3.
- The current local TTS service/container has been operated with an Edge-TTS-compatible image in recent tests.
- This metadata/runtime mismatch should be reconciled later as a separate verified change.

## 10. Development Direction

Continue under `PROJECT_RULES/DEVELOPMENT_POLICY.md`:

- one verified node/stage at a time
- full revised code for Code-node changes
- preserve known-good nodes
- isolate failures
- deterministic logic before adding new LLM calls
- local-first / free-first
- do not add engines to compensate for one unstable component
- evidence quality and factual safety take priority over filling target runtime

## 11. Immediate Next Step

The next development task is **SearXNG / Evidence Expansion search resilience**.

After that:

1. repeat the full content route without pinned data
2. verify a positive case where Expansion actually adds a new validated Fact
3. verify successful transition from `Script Validator Expanded` to `Scene Adapter`
4. then resume downstream media/end-to-end validation
