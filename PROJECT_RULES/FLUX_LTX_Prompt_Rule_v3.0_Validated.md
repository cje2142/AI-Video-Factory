# FLUX + LTX Prompt Rule v3.0 — Validated

## 1. Status
- Version: v3.0
- Status: Validated / Final backup candidate
- Pipeline: FLUX Anchor -> LTX I2V
- Validation result: 3 concept tests passed at practical-use level

## 2. Validated baseline
### FLUX
- Model: FLUX.1-schnell FP8
- Resolution: 416x736
- Current tested Schnell generation settings retained
- Role: create the stable anchor frame

### LTX
- Model: LTX 0.9.6 distilled
- Resolution: 416x736
- Frames: 97
- Steps: 8
- CFG: 1.0
- Sampler: Euler
- LTXVConditioning frame_rate: 30
- CreateVideo fps: 20
- RIFE: OFF

## 3. Algorithm-derived prompt guidance
Current LTX text path uses a T5-XXL-family text encoder through ComfyUI's LTXV CLIP/text-encoding path. In practice, the prompt should be treated as semantic natural-language conditioning rather than a rigid command parser.

### Core implications
1. Use coherent natural-language scene descriptions instead of keyword piles.
2. For LTX, place the main action early in the prompt.
3. Describe motion in a simple chronological flow.
4. Positive prompt design matters more than trying to suppress everything with a long Negative prompt.
5. Avoid directly animating fragile micro-parts such as eyelids, pupils, fingers, or lips.
6. Keep the LTX prompt consistent with what is actually visible in the FLUX anchor image.
7. For short 4–5 second clips, keep the prompt detailed but focused; avoid overloading the model with too many simultaneous actions.

## 4. Test-derived findings
### Failed / problematic directions
- Strong blink suppression such as `no visible blinking` and `extremely minimal eyelid movement` caused unnatural eye locking during head motion.
- Direct eye-motion wording such as `natural eye movement` and `subtle natural eyelid behavior` could trigger visible facial/eye reconstruction artifacts.
- Excessive motion restrictions produced stable but overly static results.
- Trying to micromanage fingers and eyes directly was less reliable than simplifying pose and composition.

### Directions that passed
- Natural scene-direction wording
- Clear framing and composition
- Face as the primary visual focus
- Hands kept low and visually secondary
- Small, natural head and upper-body movement
- Simple slow camera push-in
- Short, severe-error-focused Negative prompt
- Anchor-image state respected by the LTX prompt

## 5. Final prompt rules

### 5.1 General philosophy
- Stabilize by scene design, not by excessive prohibition.
- Be specific about composition and staging, but allow natural motion freedom.
- Describe one believable short scene as a director would.

### 5.2 Composition rules
Preferred:
- upper-body portrait or medium portrait
- balanced or centered composition
- face as main visual focus
- hands low in frame, especially near the lower center/lower third
- simple indoor background
- simple props only

Purpose:
- improve facial stability
- reduce hand prominence
- keep possible hand defects in subtitle-safe areas when practical

### 5.3 Hand rules
Preferred wording:
- hands rest naturally
- hands stay low in frame
- simple relaxed resting position
- fingers are naturally positioned
- hands remain visually secondary

Avoid:
- complex gestures
- interlaced fingers
- hands near the face
- precise finger-motion instructions
- unnecessary object manipulation

Principle:
- reduce hand complexity rather than over-commanding anatomical perfection.

### 5.4 Eye rules
Preferred wording:
- relaxed attentive gaze
- calm natural gaze
- gaze naturally follows the subtle head movement

Avoid:
- no visible blinking
- extremely minimal eyelid movement
- clear stable eyes
- natural eye movement
- subtle natural eyelid behavior
- explicit pupil/eyelid animation instructions

Principle:
- do not freeze the eyes and do not animate the eyes separately; let gaze belong to the overall head/body action.

### 5.5 Motion rules
Preferred:
- small natural head movement
- gentle upper-body shift
- subtle posture adjustment
- natural, composed, unhurried motion

Avoid:
- only one very slow movement
- perfectly still / absolutely still
- excessive stacks of stability adjectives
- abrupt or exaggerated gestures

Principle:
- guide the character toward small natural motion without making the performance rigid.

### 5.6 Camera rules
Preferred:
- slow smooth push-in
- one simple continuous camera move

Avoid:
- multiple simultaneous camera moves
- dramatic pan/tilt/zoom combinations
- fast camera motion

### 5.7 Anchor consistency rule
The LTX prompt must respect the actual starting image.
- If a hand is touching a cup in the anchor, do not say it is not touching any object.
- If the body pose differs from the planned prompt, describe the visible pose rather than forcing a contradiction.
- Use wording such as `in the same simple resting position shown in the starting image` when useful.

## 6. FLUX Positive template
```text
A single professional adult is in a clean modern indoor setting.

The shot is framed as a clear upper-body or medium portrait with a balanced composition.
The face is the main visual focus, while the hands remain low near the lower part of the frame in a simple relaxed resting position.
The fingers are naturally positioned and visually secondary to the face.

The person has a calm natural presence, a closed mouth, relaxed shoulders, and a neutral pleasant expression.
The eyes are open with a relaxed attentive gaze toward the camera.
The posture is upright but comfortable.

Photorealistic, clean lighting, sharp facial detail, natural hands, realistic proportions, uncluttered background.
```

## 7. FLUX Negative template
```text
extra fingers, missing fingers, fused fingers, deformed hands, malformed fingers,
complex hand gestures, hands near the face, awkward hand pose,
distorted eyes, asymmetrical eyes, blurry eyes,
open mouth, teeth showing, exaggerated expression,
multiple people, second person, background person, crowd,
warped anatomy
```

## 8. LTX Positive template
```text
A single professional adult makes a small natural head movement and a gentle upper-body shift in a clean modern indoor setting.

The expression remains calm, pleasant, and composed, and the relaxed attentive gaze naturally follows the movement.
The shot stays as a balanced upper-body or medium portrait with a clear composition.
The face remains the main visual focus, while the hands stay low in the frame in the same simple resting position shown in the starting image.

The posture remains comfortable and natural.
The camera slowly pushes forward in one smooth continuous motion.
The overall movement feels natural, composed, and unhurried.

Photorealistic, clean lighting, stable facial detail, natural hands, realistic continuity, uncluttered background.
```

## 9. LTX Negative template
```text
extra fingers, missing fingers, fused fingers, deformed hands,
distorted eyes, asymmetrical eyes, blurry eyes,
abrupt head turns, jerky motion, sudden movement, jitter,
open mouth, teeth showing, exaggerated expression,
motion artifacts, warped anatomy
```

## 10. Validation checklist
Before promoting a prompt variation to production, verify:
- Face remains usable throughout the clip
- No uncomfortable eye-locking behavior
- No severe early-frame face reconstruction/melting
- Hands are acceptable or safely placed in lower/subtitle-safe area
- Motion is natural enough and not overly static
- No abrupt head turns or jitter
- Anchor state and LTX text do not contradict each other
- Camera motion remains simple and stable

## 11. Validation history summary
- Early baseline: strong blink suppression improved blink control but produced unnatural eye locking during head movement.
- Intermediate test: more natural eyelid/eye wording improved freedom but one cafe test showed early facial/eye deformation.
- Final candidate rule: removed direct micro-control of eyelids/eyes, retained natural gaze tied to head motion, simplified negative prompt, clarified composition and hand placement.
- Final 3-concept validation: all three judged usable; one studio clip remained slightly static with mild noise but still passed practical-use criteria.

## 12. Backup name
Recommended filename:
`FLUX_LTX_Prompt_Rule_v3.0_Validated.md`

Recommended future versioning:
- v3.0 = validated natural-direction baseline
- v3.1 = only after a new rule change is separately tested and validated
