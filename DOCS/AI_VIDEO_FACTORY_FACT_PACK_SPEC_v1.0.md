# AI Video Factory — Fact Pack Spec v1.0

_Date: 2026-09-18_
_Status: DESIGN LOCKED / PRE-N8N IMPLEMENTATION_

## 1. Purpose

Fact Pack is a compact factual guardrail placed between Concept Grounding and the Qwen2.5 Script Writer.

Its job is not to produce an academic research report and not to rewrite the final script.

Its job is to provide:
- the normalized topic,
- a small set of usable facts,
- uncertainty boundaries,
- prohibited overclaims,
- and safe wording guidance

so the Script Writer can remain natural and expressive without inventing fatal misinformation.

## 2. Pipeline Position

```text
Pipeline Config
→ Prepare Run Folder
→ Concept Grounding
→ Fact Pack
→ Qwen2.5 14B Script Writer
→ Light Validator
→ Scene Adapter
```

## 3. Design Principles

1. Accuracy over style inside Fact Pack.
2. Compact output over exhaustive research.
3. Only include facts useful for the target Shorts script.
4. Separate confirmed facts from uncertainty.
5. Do not generate the final narration.
6. Do not generate FLUX/LTX prompts.
7. Do not add model/camera/visual-generation parameters.
8. Do not force academic phrasing on downstream writing.
9. Block fatal misinformation, not harmless simplification.
10. Prefer "unknown / uncertain" over invented certainty.

## 4. Input Contract

Fact Pack receives a single normalized object from Concept Grounding.

Recommended minimum input:

```json
{
  "topic_original": "",
  "topic_normalized": "",
  "core_intent": "",
  "content_type": "informational_short",
  "audience": "general",
  "language": "ko",
  "target_duration_sec": 40,
  "risk_domain": "general"
}
```

### Input notes

- `topic_original`: raw user topic.
- `topic_normalized`: ambiguity-resolved topic from Concept Grounding.
- `core_intent`: what the video should explain.
- `risk_domain`: one of:
  - `general`
  - `science`
  - `health`
  - `finance`
  - `legal`
  - `safety`

Fact Pack must not reinterpret the topic unless Concept Grounding is clearly malformed.

## 5. Output Contract

Fact Pack must return valid JSON only.

Recommended schema:

```json
{
  "fact_pack_version": "1.0",
  "topic_normalized": "",
  "core_intent": "",
  "risk_domain": "general",
  "facts": [
    {
      "id": "F1",
      "claim": "",
      "confidence": "HIGH",
      "script_use": "ALLOW"
    }
  ],
  "uncertainties": [
    {
      "id": "U1",
      "issue": "",
      "guidance": ""
    }
  ],
  "prohibited_claims": [
    ""
  ],
  "safe_framing": [
    ""
  ],
  "writer_notes": [
    ""
  ],
  "status": "PASS"
}
```

## 6. Field Rules

### facts

Target size:
- normal topic: 3–6 items
- complex topic: maximum 8 items

Each fact should be:
- directly useful to the script,
- concise,
- non-redundant,
- phrased as a claim rather than a full narration sentence.

`confidence`:
- `HIGH`: stable/common knowledge or strongly supported
- `MEDIUM`: reasonable but context-dependent
- `LOW`: should usually be moved to `uncertainties`

`script_use`:
- `ALLOW`: may be used directly or paraphrased
- `QUALIFY`: may be used only with softer wording
- `AVOID`: should not be used as a factual statement

Do not output a long bibliography in v1.0.

### uncertainties

Use when:
- causal mechanism is uncertain,
- exceptions are common,
- the answer varies by context,
- a popular explanation is plausible but not definitive.

The goal is not to stop the script. The goal is to stop false certainty.

### prohibited_claims

Only include claims that would create a serious factual problem.

Examples:
- unsupported medical diagnosis or treatment certainty
- unsupported scientific causation
- fabricated statistics
- guaranteed financial outcome
- claim that reverses the actual topic meaning

Do not put harmless casual wording here.

### safe_framing

Provide short wording boundaries such as:
- "보통 ~로 알려져 있다"
- "~일 가능성이 있다"
- "상황에 따라 다를 수 있다"

Use only when needed. Do not force hedging into every sentence.

### writer_notes

Maximum 3 short notes.

Use for:
- tone or audience cautions,
- key distinction the writer must preserve,
- one important "do not confuse X with Y" note.

Do not include visual-generation instructions.

## 7. Status Rule

### PASS
Return `PASS` when the topic is sufficiently grounded for script writing.

### WARN
Return `WARN` when the script can proceed but one or more claims need qualification.

### FAIL
Return `FAIL` only when:
- the normalized topic is internally contradictory,
- the topic is still too ambiguous to form a safe factual base,
- reliable factual grounding is insufficient for a high-risk claim,
- or the input itself contains a fatal misconception that cannot be safely carried forward.

FAIL should be rare.

## 8. Fact Pack Prompt — Production Draft

### System Prompt

```text
You are the Fact Pack node for a Korean informational Shorts pipeline.

Your role is factual grounding, not script writing.

Given a normalized topic and core intent:
1. identify only the few facts that are useful for the script,
2. separate uncertain or context-dependent points,
3. identify only serious claims that must not be stated,
4. provide minimal safe-framing guidance when needed.

Rules:
- Do not write narration.
- Do not write hooks.
- Do not write scene descriptions.
- Do not write image/video prompts.
- Do not add camera/model/generation parameters.
- Do not invent statistics, mechanisms, causes, or certainty.
- Prefer compact, useful facts over academic completeness.
- Harmless simplification is allowed downstream.
- Mild exaggeration or casual style is not your job to police.
- Only serious factual risk belongs in prohibited_claims.
- If a point is uncertain, put it in uncertainties instead of pretending it is certain.
- Output valid JSON only.
```

### User Prompt Template

```text
Create a Fact Pack from this normalized input.

topic_original: {{topic_original}}
topic_normalized: {{topic_normalized}}
core_intent: {{core_intent}}
content_type: {{content_type}}
audience: {{audience}}
language: {{language}}
target_duration_sec: {{target_duration_sec}}
risk_domain: {{risk_domain}}

Return exactly these fields:
fact_pack_version
topic_normalized
core_intent
risk_domain
facts
uncertainties
prohibited_claims
safe_framing
writer_notes
status
```

## 9. Deterministic Validation After LLM

A Code node should validate the Fact Pack before it reaches Script Writer.

Required checks:

```text
valid JSON
fact_pack_version exists
topic_normalized non-empty
core_intent non-empty
risk_domain valid
facts is array
facts count <= 8
every fact has id / claim / confidence / script_use
confidence ∈ HIGH|MEDIUM|LOW
script_use ∈ ALLOW|QUALIFY|AVOID
uncertainties is array
prohibited_claims is array
safe_framing is array
writer_notes is array
writer_notes count <= 3
status ∈ PASS|WARN|FAIL
no scene fields
no FLUX/LTX technical fields
no narration field
```

Forbidden technical keys include:

```text
scene
scenes
camera
framing
width
height
frames
fps
steps
cfg
sampler
scheduler
seed
checkpoint
model_name
flux_prompt
ltx_prompt
negative_prompt
```

## 10. Retry / Failure Policy

If JSON/schema validation fails:
- retry the Fact Pack LLM once with a compact correction prompt,
- do not regenerate Concept Grounding automatically.

If the second attempt still fails:
- mark `FACT_PACK_SCHEMA_FAIL`
- stop before Script Writer.

If Fact Pack returns `FAIL`:
- stop content generation and preserve the reason for review.

If Fact Pack returns `WARN`:
- continue to Script Writer with uncertainties and safe framing intact.

## 11. High-Risk Domain Adjustment

For `health`, `finance`, `legal`, or `safety`:
- be stricter about unsupported certainty,
- move weak causal claims into uncertainties,
- add explicit prohibited claims only where misuse would be serious.

Do not turn the whole script into legal/medical disclaimer language.
The downstream video should still sound natural.

## 12. What Fact Pack Must NOT Do

Fact Pack must not:
- write the final script,
- decide scene count,
- assign scene durations,
- produce subtitles,
- plan shots,
- create FLUX/LTX prompts,
- rewrite natural language for style,
- act as the final Light Validator,
- perform visual validation,
- add facts not necessary for the topic.

## 13. Example — General Topic

Input topic:
`고양이가 갑자기 집 안을 빠르게 뛰어다니는 이유`

Example compact output:

```json
{
  "fact_pack_version": "1.0",
  "topic_normalized": "고양이가 갑자기 집 안을 매우 빠르게 뛰어다니는 행동(zoomies)",
  "core_intent": "고양이의 갑작스러운 빠른 달리기 행동이 왜 나타나는지 쉽게 설명",
  "risk_domain": "general",
  "facts": [
    {
      "id": "F1",
      "claim": "고양이에게 짧고 갑작스러운 고속 달리기 행동이 나타날 수 있다.",
      "confidence": "HIGH",
      "script_use": "ALLOW"
    },
    {
      "id": "F2",
      "claim": "쌓인 에너지를 빠르게 소비하는 행동으로 설명되는 경우가 많다.",
      "confidence": "MEDIUM",
      "script_use": "QUALIFY"
    },
    {
      "id": "F3",
      "claim": "놀이, 흥분, 특정 시간대의 활동성 증가와 함께 나타날 수 있다.",
      "confidence": "MEDIUM",
      "script_use": "QUALIFY"
    }
  ],
  "uncertainties": [
    {
      "id": "U1",
      "issue": "모든 zoomies를 하나의 원인으로 설명할 수는 없다.",
      "guidance": "한 가지 이유로 단정하지 말고 여러 가능성을 자연스럽게 제시한다."
    }
  ],
  "prohibited_claims": [
    "zoomies가 특정 질병을 의미한다고 단정",
    "모든 고양이에게 동일한 원인이 있다고 단정"
  ],
  "safe_framing": [
    "보통",
    "경우가 많다",
    "상황에 따라"
  ],
  "writer_notes": [
    "우다다를 울음소리로 해석하지 말 것"
  ],
  "status": "PASS"
}
```

## 14. Acceptance Criteria

Fact Pack v1.0 is acceptable for n8n implementation when all are true:
1. output is deterministic enough for JSON parsing,
2. normal topics usually return 3–6 useful facts,
3. the node does not generate narration,
4. uncertainty is separated rather than hidden,
5. serious factual errors are blocked without over-policing style,
6. output can be consumed directly by Script Writer,
7. schema validation can be implemented with a deterministic Code node.

## 15. Next Step After Validation

After Fact Pack prompt + schema passes local Qwen2.5 testing:

```text
Fact Pack
→ Qwen2.5 Script Writer
```

The Script Writer spec should consume this exact Fact Pack structure without redefining factual authority.
