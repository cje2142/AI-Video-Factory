# AI Video Factory
## Automation Architecture Draft v0.1 (자동화 아키텍처 설계초안 v0.1)

**Version:** Draft v0.1  
**Status:** Architecture Design Draft (아키텍처 설계초안)  
**Implementation:** NOT STARTED  
**Engine Selection:** NOT FINALIZED  
**Roadmap:** NOT YET APPLIED  

## 1. Project Goal (프로젝트 목표)
사용자와 ChatGPT가 `concept`, `channel` 및 필요한 선택 항목을 정하면, 기획부터 Scene(씬) 제작, 영상 생성, 검증, 음성/자막/BGM, 최종 렌더까지 자동화하여 약 40~90초의 최종 영상을 생성한다. 업로드는 현재 자동화 범위에서 제외하며 사용자가 최종 판단한다.

```text
User + ChatGPT
→ Planning
→ Scene Production
→ Video Generation
→ Validation
→ Voice / Subtitle / BGM
→ Final Render
→ Final Validation
→ shorts_final.mp4
→ User Final Decision
```

## 2. Core Quality Priority (핵심 품질 우선순위)
1. Video Stability (영상 안정성)
2. Concept & Story Continuity (콘셉트·스토리 연속성)
3. Natural Visual Quality (자연스러운 영상 품질)

최고 해상도 자체보다 시청 중 형태 붕괴나 장면 이탈이 눈에 띄지 않는 안정성을 우선한다.

## 3. User Input Structure (사용자 입력 구조)
필수 입력:
- `concept`
- `channel`

선택 입력:
- `duration`
- `target_audience`
- `tone`
- `voice_style`
- `bgm_style`
- `quality_mode`

참고자료는 기본 필수가 아니며, Planning Stage(기획 단계)에서 AI가 필요성을 판단해 추천하고 사용자가 채택·직접 지정·무시할 수 있다.

## 4. Pipeline Config (파이프라인 중앙 설정)
기술 설정은 여러 노드에 흩어놓지 않고 중앙 설정에서 관리한다.

```text
Pipeline Config
├─ User Options (사용자 선택)
├─ Production Defaults (제작 기본값)
├─ Engine Profiles (엔진 프로필)
├─ Audio Profile (오디오 프로필)
├─ Subtitle Profile (자막 프로필)
└─ Validation Profile (검증 프로필)
```

실행 안정성이 필요한 JSON key, 변수명, 모델명, 노드명, API 필드명은 영어만 사용한다.

```json
{
  "scene_duration": 5,
  "resolution": "384x672",
  "output_fps": 12,
  "rife_multiplier": 3,
  "retry_limit": 1,
  "base_engine": "TBD",
  "fallback_engine": "TBD"
}
```

원칙:
- Config = 기술 설정
- Planner = 콘텐츠 설계
- Prompt Adapter = 모델별 명령 변환

## 5. Planning Architecture (기획 구조)
```text
Concept Input
→ Core Intent (핵심 의도)
→ Story Plan (스토리 설계)
→ Scene Plan (씬 구성)
→ Plan Validator (기획 검증기)
→ Scene Production Plan (씬 제작 계획)
```

### Core Intent (핵심 의도)
```text
topic
core_message
target_audience
tone
channel
target_duration
must_include
avoid_direction
```

### Story Plan (스토리 설계)
고정된 `Intro → Body → Conclusion` 형식을 강제하지 않는다. Hook(강한 도입), Curiosity(궁금증), Contrast(비교), Surprise(의외성), Problem(문제 제기), Reversal(반전), Payoff(결론 보상) 등을 콘셉트에 따라 자유롭게 사용한다.

### Scene Plan (씬 구성)
- Scene당 약 4~6초
- 전체 약 40~90초
- Scene 수는 전체 길이에 따라 동적으로 결정

### Scene Production Plan (씬 제작 계획)
```text
scene_id
purpose
narration
visual_concept
subject
action
environment
camera
motion
continuity
duration
```

Planner는 무엇을 만들지 결정하고, Prompt Adapter(프롬프트 변환기)는 각 엔진에 어떻게 명령할지 변환한다.

## 6. Generation Mode (생성 방식)
플랫폼은 Engine Neutral Architecture(엔진 중립 구조)로 설계한다.

### General Mode (일반 모드)
고정 캐릭터 유지가 중요하지 않은 영상용.

```text
Scene Plan
→ Prompt Adapter
→ LTX T2V
```

`LTX T2V`는 우선 테스트 후보이며 확정 엔진이 아니다.

### Character Mode (고정 캐릭터 모드)
동일 진행자·캐릭터의 일관성이 중요한 영상용.

```text
Character Reference
→ FLUX
→ Scene Image
→ LTX I2V
```

현재 역할 후보:
- FLUX = Image Generator (기준 이미지 생성기)
- LTX = I2V Video Engine (I2V 영상 엔진)

실측 테스트 전까지 확정하지 않는다.

## 7. Character Reference System (고정 캐릭터 기준 시스템)
고정 캐릭터는 Scene마다 새로 만들지 않고 `Character Master Image`와 `Character Profile`을 재사용한다.

```text
master_image
age
gender
role
hairstyle
outfit
facial_mood
stable_traits
prop_rules
forbidden_changes
```

고정 대상:
- face
- hairstyle
- identity
- outfit style
- character tone

변경 가능 대상:
- pose
- background
- camera
- gesture
- prop position
- scene environment

## 8. Engine Prompt Adapter (엔진 프롬프트 변환기)
기존 Prompt Engine의 역할을 단순화한다.

```text
Scene Production Plan
→ Engine Prompt Adapter
→ Model-specific Prompt
```

콘텐츠 의미와 모델별 제약을 분리하고, 모델 특화 제한은 Adapter에서만 관리한다.

## 9. Default / Fallback Strategy (기본·예외 엔진 전략)
Scene 복잡도만으로 예외 엔진을 선택하지 않는다.

```text
Default Engine
→ Generate
→ Validate
```

정상이면 그대로 사용한다.

```text
CRITICAL
→ Same Engine Regeneration
→ New Seed
```

다시 `CRITICAL`이면 Fallback Engine(예외·대체 엔진)으로 전환한다.

현재 후보:
- Default: LTX family candidate
- Fallback / Quality: Wan family candidate

두 후보 모두 미확정이다.

## 10. RIFE Frame Interpolation (RIFE 프레임 보간)
RIFE는 현재 테스트를 통해 채택된 공통 후처리 단계다.

```text
Video Engine
→ RIFE
→ Visual Validator
```

현재 균형 후보:
- `multiplier = 3`
- `output_fps = 12`

대안:
- x2 → 8fps
- x4 → 16fps

RIFE는 motion smoothing(동작 부드러움)과 frame interpolation(프레임 보간) 용도이며 얼굴·손 변형, identity drift(정체성 흔들림), object morphing(객체 변형) 자체를 해결하는 엔진으로 보지 않는다.

## 11. Plan Validator (기획 검증기)
영상 생성 전 다음 항목을 검사한다.

```text
concept drift
scene repetition
duration error
story discontinuity
scene-purpose mismatch
```

## 12. Visual Validator (영상 검증기)
목표는 예술 평가가 아니라 치명적 오류 탐지다.

판정:
```text
PASS
MINOR
CRITICAL
```

CRITICAL 예:
```text
severe face deformation
severe hand/body deformation
identity change
major background collapse
scene mismatch
strong flicker
broken frames
major object morphing
```

초기 Validation Sampling(검증 프레임 샘플링)은 4~6초 Scene 기준 Start / Middle / End 3 frame을 우선 사용하며 필요 시 5 frame으로 확대한다.

## 13. Retry Logic (재생성 로직)
무한 반복을 금지한다.

```text
Default Engine
→ CRITICAL
→ Regenerate Once
→ CRITICAL Again
→ Fallback Engine
```

Fallback 이후에도 무한 재생성하지 않고 필요 시 상태를 기록해 사용자 판단으로 넘긴다.

## 14. Post Production (후반 제작)
기존에 완성된 자동화 구조는 최대한 재사용한다.

```text
Scene Videos
→ Merge
→ TTS
→ Whisper
→ SRT
→ BGM
→ FFmpeg
→ Final MP4
```

### Voice System (음성 시스템)
무료·Local 우선. 선택 축은 과도하게 늘리지 않는다.
- Male / Female
- Bright / Calm / News / Friendly

### BGM System (BGM 시스템)
BGM은 필수가 아니며 작은 안전한 음악 풀을 우선한다.
- Information
- Emotional
- Tension
- Upbeat
- None

## 15. Final AV Validator (최종 음성·영상 검증기)
FFmpeg Final Render 이후 실제 최종 MP4를 검사한다.

```text
file_exists
video_duration
audio_duration
subtitle_presence
subtitle_coverage
audio_sync
bgm_volume
final_file_valid
```

가능하면 AI 판단보다 deterministic check(결정형 검사)를 우선한다.

## 16. Run Manifest (실행 기록)
복잡한 별도 시스템 대신 실행별 JSON 기록을 권장한다.

```json
{
  "run_id": "",
  "concept": "",
  "channel": "",
  "scene_count": 0,
  "generation_mode": "",
  "engine_used": "",
  "retry_count": 0,
  "validator_result": "",
  "final_file": ""
}
```

목적은 엔진별 실패율, 재생성 빈도, 안정적인 설정을 추적하는 것이다.

## 17. Current Engine Candidates (현재 엔진 후보)
| 역할 | 후보 | 상태 |
|---|---|---|
| General Mode (일반 모드) | LTX T2V | 우선 테스트 |
| Character Image (캐릭터 이미지) | FLUX Schnell 계열 | 우선 검토 |
| Character Video (캐릭터 영상) | LTX I2V | 우선 테스트 |
| Fallback / Quality (예외·고품질) | Wan 계열 | 후속 테스트 |
| Legacy / Test (기존·시험) | DreamShaper + AnimateDiff | 유지 |
| Frame Interpolation (프레임 보간) | RIFE | 채택 |

엔진 후보는 실측 검증 전까지 Architecture(아키텍처)와 분리한다.

## 18. Existing System Reuse (기존 시스템 재사용)
현재 완료된 다음 기능은 재사용을 우선한다.

```text
n8n orchestration
Run ID
Scene Split
ComfyUI API
Scene file collection
TTS
Whisper
SRT
FFmpeg merge
final output
```

AnimateDiff는 폐기하지 않고 Legacy Engine(기존 엔진), Test Engine(시험 엔진), Backup Experiment(백업 실험) 용도로 보존한다.

## 19. Complexity Control (복잡도 통제)
현재 단계에서 추가하지 않는다.

```text
Scene complexity based multi-engine routing
Multiple engines generating simultaneously
0~100 artistic scoring
Unlimited regeneration
Content-specific separate workflows
Excessive Planner node splitting
Blender integration
Automatic upload
```

새 기능은 품질 향상 효과가 구조 복잡도보다 큰지 먼저 검증한다.

## 20. Platform Philosophy (플랫폼 설계 철학)
```text
One Platform
Engine Neutral
Simple Architecture
Config Driven
Local / Free First
RTX 3060 12GB Practicality
Controlled Randomness
Critical Error Recovery Only
Human Final Decision
```

창의적 랜덤성은 살리되 콘셉트 이탈과 치명적 오류만 통제한다.

## 21. Integrated Architecture Draft (통합 설계초안)
```text
User + ChatGPT
        │
        ▼
Concept / Channel / Options
        │
        ▼
Pipeline Config
        │
        ▼
Qwen Planner
        │
        ├─ Core Intent
        ├─ Story Plan
        └─ Scene Plan
        │
        ▼
Plan Validator
        │
        ▼
Scene Production Plan
        │
        ▼
Generation Mode
   ┌────┴──────────────┐
   │                   │
General Mode       Character Mode
   │                   │
LTX T2V            Character Reference
Candidate               │
   │                  FLUX
   │                   │
   │              Scene Image
   │                   │
   │                LTX I2V
   └─────────┬─────────┘
             │
             ▼
            RIFE
             │
             ▼
      Visual Validator
             │
      ┌──────┼──────────┐
      │      │          │
    PASS   MINOR     CRITICAL
      │      │          │
      │      │     Regenerate Once
      │      │          │
      │      │     CRITICAL Again
      │      │          │
      │      │     Fallback Engine
      └──────┴──────────┘
             │
             ▼
      Post Production
             │
      TTS / Whisper
       SRT / BGM
             │
             ▼
        FFmpeg Final
             │
             ▼
      Final AV Validator
             │
             ▼
      shorts_final.mp4
             │
             ▼
      User Final Decision
```

---

This file is a design backup only. It does not lock implementation details or engine selection.