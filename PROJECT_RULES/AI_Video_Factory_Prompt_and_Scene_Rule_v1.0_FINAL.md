# AI Video Factory
# Prompt & Scene Rule v1.0 — FINAL

## 1. 목적

본 규칙은 다음 자동화 Pipeline에서 사용한다.

```text
n8n
→ Script
→ Scene Split
→ Ollama Scene Analysis
→ Scene Type Classification
→ Risk Evaluation
→ Motion Level Selection
→ FLUX Anchor Prompt
→ FLUX Image
→ LTX I2V Prompt
→ LTX Video
```

목표는 매 Scene마다 Ollama가 임의로 프롬프트를 만드는 것이 아니라,

```text
Scene Type
→ Risk
→ Motion
→ Type-specific Prompt Rule
```

순서로 일관된 생성 전략을 선택하게 하는 것이다.

---

## 2. Core Principle

### FLUX 역할

```text
정지 이미지의
Subject
Composition
Pose
Environment
Lighting
Geometry
를 확정한다.
```

FLUX에서 불필요하게 복잡한 움직임을 설명하지 않는다.

### LTX 역할

```text
FLUX Anchor의 상태를 유지하면서
Motion
Viewpoint
Perspective
Continuity
를 추가한다.
```

LTX Prompt는 반드시 실제 Anchor 이미지와 모순되지 않아야 한다.

---

## 3. Scene Type

Scene Type은 아래 **5종만 사용한다.**

```text
DRONE_NATURE
CHARACTER_3D
SCREEN_UI
PRODUCT_OBJECT
REAL_HUMAN
```

새로운 Type을 임의 생성하지 않는다.

---

## 4. Scene Type 기본값

| Scene Type | 기본 Motion | 기본 Camera/View | 사용 상태 |
|---|---|---|---|
| DRONE_NATURE | HIGH | FORWARD_RISE | PASS |
| CHARACTER_3D | MEDIUM | FOLLOW_OR_STATIC | PASS |
| SCREEN_UI | LOW | SUBTLE_VIEWPOINT | PASS |
| PRODUCT_OBJECT | MEDIUM | VIEWPOINT_SHIFT | PASS |
| REAL_HUMAN | LOW | SLOW_PUSH_OR_STATIC | CONDITIONAL PASS |

---

## 5. Scene Classification Rule

한 Scene에 여러 요소가 존재하면 다음 3가지를 평가한다.

```text
1. 화면에서 가장 중요한 피사체인가?
2. Scene의 의미를 전달하는 핵심인가?
3. 실제 움직임의 주체인가?
```

3개 중 **2개 이상**을 만족하는 대상을 Primary Subject로 결정하고 Scene Type을 선택한다.

### 대표 복합 Scene

#### 사람 + 모니터

```text
사람의 행동/표정이 핵심
→ REAL_HUMAN

차트/데이터가 핵심
→ SCREEN_UI
```

#### 사람 + 제품

```text
사람이 핵심
→ REAL_HUMAN

제품 외형이 핵심
→ PRODUCT_OBJECT
```

#### 3D 캐릭터 + 자연

```text
캐릭터가 행동의 주체
→ CHARACTER_3D

캐릭터가 매우 작고 풍경이 주제
→ DRONE_NATURE
```

#### 제품 + UI

```text
제품 디자인이 핵심
→ PRODUCT_OBJECT

화면 정보/그래프가 핵심
→ SCREEN_UI
```

---

## 6. Motion Level

Motion Level은 아래 3개만 사용한다.

```text
LOW
MEDIUM
HIGH
```

### LOW

적합:

```text
REAL_HUMAN
SCREEN_UI
High-risk Scene
```

예:

```text
small head movement
gentle upper-body shift
subtle posture change
slow viewpoint change
slow push
```

### MEDIUM

적합:

```text
PRODUCT_OBJECT
CHARACTER_3D
```

예:

```text
viewpoint shift
perspective change
body sway
simple dance
simple full-body action
moderate character motion
```

### HIGH

적합:

```text
DRONE_NATURE
Low-risk CHARACTER_3D
```

예:

```text
forward movement
rising movement
wide environmental movement
active full-body character animation
```

---

## 7. CHARACTER_3D Motion 승격 규칙

`CHARACTER_3D` 기본값은 `MEDIUM`.

다음 조건이 모두 만족될 경우에만 `HIGH` 허용.

```text
stylized character
face not close-up
hands/paws not close-up
simple anatomy
full-body or medium-wide composition
low detail risk
```

하나라도 위험도가 높으면 `MEDIUM` 유지.

---

## 8. Risk Engine

모든 Scene은 아래 세 Risk를 평가한다.

```json
{
  "human_risk": "NONE | LOW | HIGH",
  "detail_risk": "LOW | MEDIUM | HIGH",
  "motion_risk": "LOW | MEDIUM | HIGH"
}
```

### human_risk

```text
실사 사람 없음 → NONE
3D / stylized character → LOW
실사 사람 → HIGH
```

### detail_risk

다음 요소가 많을수록 상승한다.

```text
face close-up
finger close-up
complex hands
small text
precise logo
fine mechanical parts
complex geometry
```

### motion_risk

다음 요소가 많을수록 상승한다.

```text
fast dance
jump
spin
rapid hand movement
large facial expression change
complex interaction
multiple moving subjects
```

---

## 9. Global Risk Rule

핵심 안전 규칙:

```text
Risk ↑
→ Motion ↓
→ Shot simpler
→ Prompt simpler
```

Risk가 높다고 프롬프트를 더 복잡하게 작성하지 않는다.

### 자동 Motion Downshift

```text
motion_risk HIGH
→ Motion Level 최대 MEDIUM

REAL_HUMAN + motion_risk HIGH
→ Motion Level LOW

detail_risk HIGH + REAL_HUMAN
→ Motion Level LOW

CHARACTER_3D + detail_risk HIGH
→ HIGH 금지
```

---

## 10. DRONE_NATURE Rule

### 적합 대상

```text
forest
mountain
river
lake
ocean
valley
cliff
field
wide environment
drone view
```

### FLUX 핵심

```text
wide composition
foreground
midground
background
clear depth
natural light
strong landscape layers
```

### LTX 핵심

기본 모션:

```text
forward glide
+
gradual rise
```

필수 개념:

```text
reveal
parallax
depth
foreground movement
distant stable background
```

카메라 동작은:

```text
Primary Motion 1
+
Secondary Motion 1
```

까지만 기본 허용.

예:

```text
forward + rise
```

전진 + 상승 + 회전 + tilt 같은 다중 모션은 피한다.

### Template

#### FLUX

```text
A cinematic aerial view of {environment} at {time_of_day}.

The scene contains {terrain_elements} with clear foreground, midground, and background layers.
The composition is wide, balanced, and visually deep.
{atmospheric_elements} add natural depth and scale.

Photorealistic cinematic landscape, realistic aerial perspective, detailed natural textures, natural lighting.
```

#### LTX

```text
The viewpoint moves smoothly forward over {environment}, following {visual_path}.

As the viewpoint continues forward, it gradually {rise_or_descend}, revealing {distant_landscape}.
Foreground elements move faster than the distant scenery, creating clear natural parallax.

The movement remains continuous and immersive with stable landscape geometry.
Photorealistic aerial footage, realistic atmospheric depth and natural motion.
```

---

## 11. CHARACTER_3D Rule

### 적합 대상

```text
3D character
animal character
anthropomorphic character
stylized human
cute character
animated character
```

### 핵심

```text
face not excessively large
hands/paws simple
clear silhouette
space for movement
medium or full-body composition preferred
```

세밀한 손가락 동작보다 **전체 동작의 가독성**을 우선한다.

### Template

#### FLUX

```text
A visually appealing 3D {character_type} character is in {environment}.

The character is shown in a clear {shot_size} composition with enough space for movement.
The face is expressive and appealing, while the hands or paws remain simple and clearly separated.
The character wears {clothing_or_accessories}.

The pose is balanced and ready for {intended_action}.
High-quality 3D character style, polished textures, clean lighting, appealing proportions.
```

#### LTX

```text
The 3D {character_type} character performs {action} in {environment}.

The movement includes {body_motion}, {limb_motion}, and {head_motion}.
The action is lively, readable, playful, and naturally coordinated.

The character maintains stable proportions and a clear silhouette throughout the movement.
High-quality 3D animation with smooth character motion.
```

---

## 12. SCREEN_UI Rule

### 적합 대상

```text
monitor
dashboard
chart
graph
data screen
UI
analytics display
```

### 검증 결과

```text
chart structure → usable
graph structure → usable
panel layout → usable

small exact text → unreliable
```

따라서 문자 정확도는 핵심 평가 대상에서 제외한다.

### 핵심

```text
screen geometry stable
chart anchored
panels anchored
minimal UI animation
small text minimized
```

### Template

#### FLUX

```text
A realistic {device_type} displays a clean professional {ui_type} interface.

The screen is the primary visual focus.
The interface contains {main_chart}, {secondary_chart}, and simple rectangular information panels.
The layout is clean, geometric, organized, and visually balanced.

Small detailed text is minimized while chart and panel structure remain visually dominant.
Photorealistic display, crisp graphics, clean UI geometry.
```

#### LTX

```text
The {device_type} remains stable while the displayed {ui_type} interface maintains its organized structure.

The charts and interface panels stay anchored in their original positions.
The screen geometry and panel alignment remain consistent.

A subtle viewpoint change creates a natural perspective shift without disturbing the interface structure.
Photorealistic display footage with stable screen geometry and clean chart continuity.
```

---

## 13. PRODUCT_OBJECT Rule

### 적합 대상

```text
smartphone
car
cup
perfume
electronics
machine
chip
product
object close-up
```

### 검증 결과

제품 형태와 디테일은 안정적이었으나,

```text
camera motion
```

이라는 표현이 제품 자체의 카메라 부품이나 광고 모션그래픽으로 잘못 해석될 수 있었다.

따라서 `PRODUCT_OBJECT`에서는 기본적으로 **camera motion이라는 표현을 사용하지 않는다.**

### 핵심 표현

```text
object remains fixed
viewpoint shifts
perspective changes
visible parallax
```

### 필수 금지

```text
exploded view
floating parts
separated components
disassembly
floating text
text overlay
```

### Template

#### FLUX

```text
A premium {product} rests on {surface} in {environment}.

The product is clearly visible from a {view_angle} angle.
Its shape, edges, materials, and key design details are clearly defined.

The scene contains foreground, midground, and background depth so perspective changes can be perceived clearly.
Photorealistic product photography, realistic materials, precise geometry, clean reflections.
```

#### LTX

```text
The {product} remains completely fixed in place.

The viewpoint gradually shifts {viewpoint_direction}, creating a clear change in perspective around the stationary object.
The visible angle changes naturally while the shape, edges, and structural details remain consistent.

Foreground and background elements move relative to the object, producing visible parallax.
The motion comes only from the changing viewpoint.

Photorealistic product footage, smooth perspective shift, stable geometry and realistic reflections.
```

---

## 14. REAL_HUMAN Rule

### 검증 결과

실사 사람은:

```text
static / low motion → usable
high motion → unstable
```

특히 취약 영역:

```text
fingers
eyes
blinking
facial expressions
fast dance
large gestures
```

### 허용

```text
small natural head movement
gentle upper-body shift
subtle posture adjustment
simple resting hands
natural gaze
```

### 금지 원칙

눈을 직접 미세 제어하지 않는다.

다음 표현은 사용하지 않는다.

```text
no visible blinking
minimal eyelid movement
natural eye movement
subtle eyelid behavior
fixed eyes
pupil movement
```

대신:

```text
the gaze naturally follows the movement
```

정도만 허용한다.

### Template

#### FLUX

```text
A photorealistic adult {person_description} is in {environment}.

The shot is framed as a clear {shot_size} portrait with a balanced composition.
The face is the main visual focus while the hands remain in a simple natural position.

The person has {expression}, relaxed shoulders, and a comfortable natural posture.
Photorealistic skin texture, realistic anatomy, clean lighting and natural proportions.
```

#### LTX

```text
The photorealistic adult {person_description} makes a small natural head movement and a gentle upper-body shift in {environment}.

The expression remains {expression}, and the gaze naturally follows the movement.
The posture remains relaxed and balanced.
The hands stay in the same simple position shown in the starting image.

The movement remains natural, composed, and restrained.
Photorealistic human footage with stable facial detail and realistic continuity.
```

---

## 15. Negative Prompt Architecture

Negative Prompt는 매 Type마다 긴 문장을 반복하지 않는다.

구조:

```text
COMMON_NEGATIVE
+
TYPE_NEGATIVE
```

### COMMON_NEGATIVE

```text
severe distortion, warped geometry, duplicated parts,
melting structure, heavy flicker, severe motion artifacts,
low detail
```

### DRONE_NATURE_NEGATIVE

```text
distorted horizon, melting terrain,
jerky movement, sudden turns, rapid spinning
```

### CHARACTER_3D_NEGATIVE

```text
extra limbs, missing limbs, duplicated limbs,
deformed hands or paws, distorted face,
unstable silhouette
```

### SCREEN_UI_NEGATIVE

```text
melting interface, drifting panels,
broken chart lines, duplicated charts,
distorted bars, unstable screen geometry
```

### PRODUCT_OBJECT_NEGATIVE

```text
exploded view, floating parts, separated components,
disassembly, duplicated components,
object movement, object rotation,
floating labels, text overlays
```

### REAL_HUMAN_NEGATIVE

```text
extra fingers, fused fingers, deformed hands,
distorted face, asymmetrical eyes,
rapid facial changes, abrupt head turns,
large gestures, fast dancing,
complex hand motion, face morphing
```

---

## 16. Fallback Rule

분류가 확실하지 않을 경우 다음 규칙을 사용한다.

```text
실사 사람이 핵심
→ REAL_HUMAN / LOW

UI 또는 차트가 핵심
→ SCREEN_UI / LOW

사람 없음 + 제품/사물 핵심
→ PRODUCT_OBJECT / MEDIUM

3D/동물 캐릭터가 핵심
→ CHARACTER_3D / MEDIUM

환경 자체가 핵심
→ DRONE_NATURE / HIGH
```

불확실한 경우 **더 높은 Motion을 선택하지 않는다.**

---

## 17. Prompt Construction Order

### FLUX

```text
1. Primary Subject
2. Environment
3. Composition / Shot
4. Pose / Geometry
5. Visual Style
6. Lighting
7. Quality
```

### LTX

```text
1. Main Action / Viewpoint Motion
2. Subject Continuity
3. Secondary Motion
4. Perspective / Parallax
5. Environment Response
6. Style / Quality
```

LTX는 **주요 움직임을 Prompt 앞부분에 배치**한다.

---

## 18. Prompt Complexity Rule

짧은 영상에서는 너무 많은 Action을 동시에 넣지 않는다.

```text
Main Action: 1
Secondary Action: 0~1
Viewpoint Motion: 0~1
```

Risk가 높을수록 요소 수를 줄인다.

---

## 19. Anchor Consistency Rule

LTX Prompt는 FLUX Anchor에서 실제로 보이는 상태를 기준으로 작성한다.

예:

```text
Anchor에서 손이 컵을 잡고 있다
→ 손이 아무것도 잡지 않는다고 쓰지 않는다.

Anchor에서 사람이 앉아 있다
→ standing이라고 쓰지 않는다.
```

필요하면:

```text
in the same position shown in the starting image
```

를 사용한다.

---

## 20. Ollama Output Schema

Ollama는 Scene마다 아래 JSON 형태로 반환한다.

```json
{
  "scene_type": "PRODUCT_OBJECT",
  "motion_level": "MEDIUM",
  "camera_mode": "VIEWPOINT_SHIFT",

  "human_risk": "NONE",
  "detail_risk": "LOW",
  "motion_risk": "LOW",

  "primary_subject": "smartphone",
  "secondary_subject": null,

  "environment": "dark studio",
  "style": "photorealistic",
  "shot_size": "medium close-up",

  "flux_positive": "...",
  "flux_negative": "...",

  "ltx_positive": "...",
  "ltx_negative": "..."
}
```

---

## 21. Validation Status

현재 실제 테스트 기준:

```text
DRONE_NATURE
PASS

CHARACTER_3D
PASS

SCREEN_UI
PASS

PRODUCT_OBJECT
PASS

REAL_HUMAN
CONDITIONAL PASS
```

---

## 22. Production Priority

영상 Scene을 선택할 수 있다면 우선순위:

```text
1. DRONE_NATURE
2. CHARACTER_3D
3. SCREEN_UI
4. PRODUCT_OBJECT
5. REAL_HUMAN
```

단, 이는 콘텐츠 중요도가 아니라 **현재 LTX 안정성 기준**이다.

---

## 23. Final Core Rule

전체 규칙을 가장 짧게 압축하면:

```text
Classify Scene
→ Evaluate Risk
→ Select Motion
→ Apply Type Rule
→ Generate FLUX Anchor
→ Describe only compatible LTX motion
```

그리고:

```text
Risk ↑
→ Motion ↓
→ Scene simpler
→ Prompt simpler
```

---

## Version

```text
AI Video Factory
Prompt & Scene Rule v1.0
Status: FINAL / VALIDATED BASELINE
```

### 이번 최적화 결과

이 버전에서는 기존 초안에 있던 **Scene Type 설명 / Motion 규칙 / Risk 규칙 / Negative Prompt / Fallback의 중복을 제거**했고, 자동화에서 애매했던 `MEDIUM_HIGH` 같은 중간 상태도 없앴다.

따라서 이 버전이면 **규칙 자체는 닫아도 되는 수준**이다. 이후 실제 n8n + Ollama 구현 과정에서 발견되는 문제는 v1.0을 계속 수정하기보다, 검증된 변경만 모아서 **v1.1**로 올리는 방식을 사용한다.
