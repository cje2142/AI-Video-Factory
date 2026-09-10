# AI Video Factory
## Final Test Roadmap v1 (최종 테스트 로드맵 v1)

**Version:** Roadmap v1  
**Status:** TEST ROADMAP (테스트 로드맵)  
**Architecture Base:** Draft v0.1  
**Implementation:** NOT STARTED  
**Engine Selection:** NOT FINALIZED  

## 1. Purpose (목적)
이 문서는 `Architecture Draft v0.1`을 실제 구현으로 옮기기 전에 필요한 테스트 순서, 통과 기준, 중단 조건, 중복 테스트 방지 원칙을 정의한다.

핵심 원칙은 다음과 같다.

```text
Test only what is necessary
Reuse previous test assets
Do not over-tune
Prefer clear pass/fail decisions
Keep the architecture engine-neutral
```

설명 문서에서는 전문 용어를 English(한글) 병기하되, JSON key, 변수명, 모델명, 노드명, API 필드명, 파일 경로 등 실행 안정성과 관련된 식별자는 영어만 사용한다.

---

## 2. Test Roadmap Overview (테스트 로드맵 개요)

| Stage | Test | 목적 | 핵심 판정 |
|---|---|---|---|
| T0 | Current Baseline (현재 기준선) | 현재 AnimateDiff 성능 확보 | 이후 엔진 비교 기준 |
| T1 | LTX T2V | General Mode (일반 모드) 기본 후보 검증 | 안정성·지시 이행·속도 |
| T2 | FLUX → LTX I2V | Character Mode (고정 캐릭터 모드) 검증 | 캐릭터 일관성·안정성 |
| T3 | Wan Fallback (조건부) | 예외·대체 엔진 가치 검증 | LTX 실패 Scene 복구 여부 |
| T4 | Engine Decision (엔진 결정) | 실제 운영 엔진 역할 확정 | Default / Character / Fallback |
| T5 | Core Pipeline (핵심 파이프라인) | Config·Planner·Adapter 연결 | 입력 → 생성 명령 정상 |
| T6 | Validation & Recovery (검증·복구) | Validator·Retry·Resume 검증 | 오류 자동검출·부분 재실행 |
| T7 | Post Production (후반 제작) | 음성·자막·BGM·FFmpeg 통합 | 최종 영상 정상 제작 |
| T8 | End-to-End (전체 통합) | 최초 완전 자동화 검증 | Concept → MP4 원클릭 |
| T9 | Multi-Concept Stability (다중 콘셉트 안정성) | 범용성 최종 검증 | 서로 다른 콘텐츠에서도 안정 |
| T10 | Production Baseline (운영 기준선) | 운영 버전 확정 | 설정·워크플로우 백업 |

---

## 3. Global Test Rules (공통 테스트 규칙)

### 3.1 Test Reuse Principle (테스트 재사용 원칙)
이전 단계에서 생성한 동일 Scene, Prompt, 실패 사례를 다음 단계 비교에도 우선 재사용한다.

```text
T0 AnimateDiff Scene
→ T1 LTX comparison

T1 CRITICAL Scene
→ T3 Wan Fallback test

T2 Character Scene
→ T9 final stability comparison where useful
```

불필요하게 새로운 테스트 영상을 반복 생성하지 않는다.

### 3.2 Stop Condition (중단 조건)
한 엔진당 기본 테스트 1회 + 보정 테스트 최대 1회까지만 허용한다.

```text
3~5 representative scenes
→ result is clear
→ stop tuning
```

판단 기준:

```text
Clearly better
→ keep as candidate
→ move to next stage

Clearly worse
→ stop tuning
→ review another candidate

Ambiguous
→ allow one settings adjustment
→ retest once
```

추가 미세튜닝은 체감 품질 개선이 명확할 때만 허용한다.

### 3.3 Resource Measurement (자원 측정)
별도 테스트 단계로 분리하지 않고 각 엔진 테스트 시 함께 기록한다.

```text
Generation Time
VRAM Peak
RAM Usage when meaningful
Failure / OOM status
```

### 3.4 Human Review Load (사용자 검토 피로도)
사용자가 영상을 반복 시청·비교해야 하는 무거운 테스트는 가능하면 다음 네 구간으로 제한한다.

```text
T0 Current Baseline
T1 LTX T2V
T2 FLUX → LTX I2V
T9 Multi-Concept Stability
```

T3는 조건부이며 T4~T8은 기능·통합 검증 중심으로 진행한다.

---

## 4. T0 — Current Baseline (현재 기준선)
목적은 현재 `DreamShaper + AnimateDiff + RIFE` 시스템의 비교 기준을 남기는 것이다.

개선이나 재튜닝은 하지 않는다.

기록 항목:

```text
Generation Time
VRAM Peak
Prompt Following
Visual Stability
Morphing / Deformation
RIFE Result
```

정성 평가는 단순하게 유지한다.

```text
GOOD
ACCEPTABLE
BAD
```

T0는 합격/불합격 단계가 아니라 T1 이후 비교를 위한 Baseline(기준선)이다.

---

## 5. T1 — LTX T2V Test
목적은 LTX T2V가 General Mode (일반 모드)의 기본 엔진 후보 자격이 있는지 검증하는 것이다.

대표 Scene 3~5개만 사용한다.

권장 범주:

```text
human
animal or living subject
object / environment
dynamic motion
general information scene
```

평가 항목:

```text
Video Stability
Prompt Following
Motion Naturalness
Morphing / Deformation
Generation Time
VRAM Usage
```

### T1 Pass Criteria (통과 기준)

```text
CRITICAL scenes ≤ 20%
Prompt Following ≥ 75%
Stability GOOD + ACCEPTABLE ≥ 80%
Motion BAD ≤ 20%
Generation Time ≤ about 2x current baseline
```

추가 필수 조건:

> AnimateDiff 대비 실제 체감 품질이 명확히 좋아야 한다.

수치가 비슷해도 체감 개선이 없으면 기본 엔진 후보로 확정하지 않는다.

### T1 CRITICAL examples

```text
severe face deformation
severe hand/body deformation
major object morphing
major background collapse
large frame-to-frame form jump
main subject identity replacement
```

---

## 6. T2 — FLUX → LTX I2V Test
목적은 Character Mode (고정 캐릭터 모드)의 실효성을 검증하는 것이다.

기본 구조:

```text
Character Master
→ FLUX Scene Image
→ LTX I2V
→ RIFE
```

동일 캐릭터 기준으로 3~5 Scene을 사용한다.

평가 항목:

```text
Identity Consistency
Face / Hair / Outfit Consistency
Scene Variation
Motion Naturalness
Visual Stability
Generation Time
```

### T2 Pass Criteria (통과 기준)

```text
Identity Consistency ≥ 80%
CRITICAL Character Drift ≤ 20%
Motion GOOD + ACCEPTABLE ≥ 80%
Scene Variation = acceptable
Character consistency clearly better than LTX T2V
```

중요 원칙:

> FLUX 정지 이미지 품질 자체가 아니라 최종 영상에서 캐릭터 일관성이 실제로 좋아지는지가 핵심이다.

`FLUX → LTX I2V`는 추가 생성 시간과 구조 복잡도를 감수할 만큼 일관성 개선이 명확할 때만 Character Mode (고정 캐릭터 모드)로 채택한다.

Character Drift (캐릭터 흔들림) 실패 예:

```text
different-looking person
major hairstyle change
major age change
complete outfit style change
major facial identity change
```

---

## 7. T3 — Wan Fallback Test (조건부)
Wan은 필수 채택 엔진이 아니다.

T1/T2에서 LTX가 충분히 안정적이고 1회 재생성으로 대부분 복구되면 T3는 축소하거나 생략할 수 있다.

새 Scene을 만들지 않고 기존 LTX 실패 Scene을 재사용한다.

```text
LTX CRITICAL Scene
→ Wan
→ compare recovery
```

평가 항목:

```text
Quality Gain
Recovery Capability
Generation Time
VRAM Burden
Automation Complexity
```

복구 이득이 작거나 생성 비용·복잡도가 과도하면 `fallback_engine = none`도 허용한다.

---

## 8. T4 — Engine Decision (엔진 결정)
새 영상을 생성하지 않는다.

T0~T3 결과만 비교해 운영 역할을 결정한다.

예상 후보 구조:

```text
General Mode
→ LTX T2V candidate

Character Mode
→ FLUX → LTX I2V candidate

Fallback
→ Wan candidate or none
```

이 단계에서 License Check (라이선스 검증)를 1회 수행한다.

검증 대상:

```text
commercial use eligibility
model license restrictions
redistribution restrictions if relevant
service / monetization compatibility
```

실제 테스트 결과와 라이선스 검증 전까지 엔진 역할을 확정하지 않는다.

---

## 9. T5 — Core Pipeline (핵심 파이프라인)
엔진 역할 결정 후 n8n 구조를 본격적으로 통합한다.

```text
Pipeline Config
→ Qwen Planner
→ Core Intent
→ Story Plan
→ Scene Plan
→ Plan Validator
→ Scene Production Planner
→ Engine Prompt Adapter
```

같은 단계에서 다음을 함께 구현·검증한다.

```text
Mode Decision
Reference Material Flow
Run Manifest
```

### Pipeline Config (파이프라인 중앙 설정)
기술값은 중앙 설정에서 관리한다.

```text
User Options
Production Defaults
Engine Profiles
Audio Profile
Subtitle Profile
Validation Profile
```

### Reference Material Flow (참고자료 흐름)
기본 입력으로 강제하지 않는다.

Planning Stage (기획 단계)에서 필요할 때만 AI가 참고자료 사용을 추천한다.

```text
Recommend
Use user-provided reference
Ignore
```

모두 허용한다.

### Run Manifest (실행 기록)
실행별 최소 상태를 기록한다.

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

---

## 10. T6 — Validation & Recovery (검증·복구)
생성·후처리 이후 자동 오류 탐지와 부분 복구를 검증한다.

```text
Generation
→ RIFE
→ Visual Validator
→ PASS / MINOR / CRITICAL
```

### Validation Sampling (검증 프레임 샘플링)
별도 단계로 두지 않는다.

4~6초 Scene 기준 초기값:

```text
Start Frame
Middle Frame
End Frame
```

3 frame으로 시작하고 오류 누락이 의미 있게 확인될 때만 5 frame으로 확대한다.

### Retry Logic (재생성 로직)

```text
Default Engine
→ CRITICAL
→ Regenerate Once with new seed
→ CRITICAL Again
→ Fallback Engine if available
```

무한 재생성은 금지한다.

### Resume / Recovery (부분 재시작·복구)
이미 성공한 Scene을 다시 생성하지 않는다.

예:

```text
Scene 1~7 complete
Scene 8 failed
→ resume from Scene 8
```

이 기능은 실제 운영 효율을 위한 필수 항목으로 본다.

---

## 11. T7 — Post Production (후반 제작)
기존 자동화 자산을 최대한 재사용한다.

```text
Scene Merge
→ TTS
→ Whisper
→ SRT
→ BGM
→ FFmpeg
→ Final AV Validator
```

검증 범위:

```text
voice generation
subtitle completeness
voice/subtitle sync
BGM presence when enabled
BGM volume balance
final render success
```

Voice Engine (음성 엔진)과 BGM Engine (BGM 엔진)은 별도 장기 테스트로 쪼개지 않고 이 단계 안에서 최소 비교 후 선택한다.

무료·Local 우선 원칙을 유지한다.

---

## 12. T8 — End-to-End Test (전체 통합 테스트)
전체 파이프라인을 처음부터 끝까지 1회 실행한다.

```text
Concept + Channel
→ Planning
→ Generation
→ Validation
→ Post Production
→ Final Validation
→ shorts_final.mp4
```

핵심 판정:

```text
one-click completion
no manual repair required for normal case
correct scene count
correct output duration
valid final MP4
```

이 단계에서는 다시 영상 엔진의 세부 화질 미세튜닝을 하지 않는다.

---

## 13. T9 — Multi-Concept Stability Test (다중 콘셉트 안정성 테스트)
범용 플랫폼으로서의 최종 안정성을 확인한다.

서로 성격이 다른 약 4개 콘셉트를 권장한다.

```text
information-focused
fixed-character
animal / story
emotional / environment
```

평가 항목:

```text
Concept Drift
Scene Continuity
Critical Error Rate
Retry Count
Final Completion
Total Production Time
```

이 단계에서 처음으로 전체 시스템의 범용성과 반복 안정성을 최종 판단한다.

---

## 14. T10 — Production Baseline (운영 기준선)
새 테스트는 하지 않는다.

통과한 설정과 워크플로우만 운영 기준선으로 확정한다.

백업 대상:

```text
Engine Profile
Pipeline Config
Planner
Prompt Adapter
Validator
RIFE
Post Production
Run Manifest
n8n Workflow
ComfyUI Workflow
```

완료 후 GitHub Backup (깃허브 백업)을 수행한다.

---

## 15. Final Decision Rules (최종 판정 원칙)

### Adopt (채택)
체감 품질 또는 안정성 개선이 명확하고 운영 복잡도가 허용 가능한 경우.

### Reject (탈락)
현재 Baseline 대비 개선이 없거나 자원·시간·복잡도 증가가 품질 이득보다 큰 경우.

### Conditional (조건부)
특정 Mode 또는 Fallback 용도로만 가치가 있는 경우.

---

## 16. Roadmap Flow (최종 흐름)

```text
T0 Baseline
    ↓
T1 LTX T2V
    ↓
T2 FLUX → LTX I2V
    ↓
T3 Wan (only if needed)
    ↓
T4 Engine Decision
════════════════════
Engine Validation Complete
════════════════════
    ↓
T5 Core Pipeline
    ↓
T6 Validation / Recovery
    ↓
T7 Post Production
    ↓
T8 End-to-End
    ↓
T9 Multi-Concept Stability
    ↓
T10 Production Baseline
```

---

## 17. Completion Criteria (완료 기준)
Roadmap v1은 다음이 모두 충족되면 완료로 본다.

```text
General Mode path validated
Character Mode path validated if adopted
Fallback path validated only if needed
No unnecessary repeated testing
One retry maximum before fallback
Resume from failed scene works
Final MP4 produced automatically
Multi-concept stability acceptable
Production Baseline backed up
```

이 문서는 구현 과정에서 테스트 결과에 따라 수정될 수 있으며, 미검증 엔진 후보를 고정 규칙으로 취급하지 않는다.
