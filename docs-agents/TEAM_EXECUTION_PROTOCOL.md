# TEAM_EXECUTION_PROTOCOL.md

Team 단위 수행 시 팀 간 핸드오프, 결과물 관리, 검증 프로세스를 정의한다.

---

## 1. Team 수행 프로세스

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          Team 수행 프로세스                                │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  [사용자 프롬프트]                                                         │
│       │                                                                  │
│       ▼                                                                  │
│  STEP 1. prompt.md 저장 (원문 그대로, 절대 가공 금지)                       │
│       │  result-docs/{prompt-id}/prompt.md                               │
│       │                                                                  │
│  ┌──► ▼                                                                  │
│  │  STEP 2. 01_planning-team → task_prompt.md 생성                       │
│  │    │  result-docs/{prompt-id}/task_prompt.md                          │
│  │    │  프롬프트 분석, 적용 팀 결정, 수행 순서, 검증 기준 수립              │
│  │    │                                                                  │
│  │    ▼                                                                  │
│  │  STEP 3. Team 순차/병렬 수행                                           │
│  │    │  각 팀 → result_{팀명}.md 생성                                    │
│  │    │  다음 팀은 prompt.md + task_prompt.md + 이전 result.md 전체        │
│  │    │  + AGENT_TEAM_STRATEGY.md에 따른 docs-claude 문서 읽기            │
│  │    │                                                                  │
│  │ ┌► ▼                                                                  │
│  │ │ STEP 4. 04_qa-team → 검증                                           │
│  │ │  │  task_prompt.md + prompt.md 기준 검증                             │
│  │ │  │  result_VERIFICATION.md에 Round별 기록                            │
│  │ │  │                                                                  │
│  │ │  ├── 이상있음 → STEP 3으로 (해당 팀 재수행)                           │
│  │ │  │                                                                  │
│  │ └──┘                                                                  │
│  │    │                                                                  │
│  │    ├── 이상없음 → STEP 2로 (01_planning-team 재분석)                    │
│  │    │                                                                  │
│  └────┘                                                                  │
│       │                                                                  │
│       ├── 이상없음 연속 2회 → 검증 완료                                    │
│       │                                                                  │
│       ▼                                                                  │
│  STEP 5. Build & Test (dev 모드)                                         │
│       │  type-check → lint → test → build (개발 환경)                     │
│       │  실패 시 코드 수정 후 STEP 3으로                                   │
│       │                                                                  │
│       ▼                                                                  │
│  완료                                                                    │
│                                                                          │
│  ※ 최대 반복 횟수: 20회 (STEP 2~4 합산)                                   │
│  ※ 배포 프로세스: 현재 미정                                                │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 2. prompt-id 규칙

- 형식: `{YYYYMMDD}_{순번}_{키워드}`
- 예시: `20260311_001_dashboard_api`
- 키워드는 프롬프트 핵심 내용을 3단어 이내로 요약
- 순번: `result-docs/` 디렉토리 내 동일 날짜의 기존 폴더를 조회하여 다음 번호를 부여한다

---

## 3. result-docs 디렉토리 구조

```
01_docs/docs-agents/result-docs/{prompt-id}/
├── prompt.md                    ← 사용자 프롬프트 원문 (절대 가공 금지)
├── task_prompt.md               ← 01_planning-team이 생성한 수행 프로세스
├── result_{팀명}.md              ← 각 팀별 결과
├── result_{팀명}-{대상}.md       ← 병렬 수행 시 대상별 결과
└── result_VERIFICATION.md       ← 04_qa-team의 검증 결과
```

### result 파일 명명 규칙

| 상황 | 파일명 | 예시 |
|------|--------|------|
| 팀이 단일 산출물 생성 | `result_{팀명}.md` | `result_03_development-team.md` |
| 팀 내 병렬 수행으로 대상별 산출물 | `result_{팀명}-{대상}.md` | `result_03_development-team-webadmin.md` |
| 검증 결과 | `result_VERIFICATION.md` | (고정) |

---

## 4. 각 파일 작성 규칙

### 4.1 prompt.md

사용자 프롬프트 원문을 **그대로** 저장한다. **절대 가공하지 않는다.** 요약, 정리, 재구성, 번역 등 어떠한 변환도 금지한다.

```markdown
# Prompt

{사용자 프롬프트 원문 — 그대로 복사}

---
저장일시: {YYYY-MM-DD HH:mm}
```

### 4.2 task_prompt.md

**01_planning-team**이 prompt.md를 분석하여 수행 프로세스를 생성한다.

```markdown
# Task Prompt

## 1. 프롬프트 분석
{요구사항 분석 내용}

## 2. 적용 Team 및 수행 범위
| 순서 | Team | 참여 에이전트 | 수행 범위 |
|------|------|-------------|-----------|
| 1 | {팀명} | {필수/보조 에이전트} | {범위} |
| 2 | {팀명} | {필수/보조 에이전트} | {범위} |
| ... | ... | ... | ... |

## 3. 수행 순서
| 순서 | Team | 수행 내용 | 참조 docs-claude | 병렬 가능 |
|------|------|----------|-----------------|----------|
| 1 | {팀명} | {내용} | {문서 목록} | N |
| 2 | {팀명} | {내용} | {문서 목록} | N |
| ... | ... | ... | ... | Y/N |

## 4. 병렬 처리 계획
{병렬 가능한 단계 식별 및 그룹핑}

## 5. 검증 기준
{04_qa-team이 검증할 항목 목록}

## 6. 빌드/테스트 계획
{빌드 명령어, 테스트 범위 — dev 모드 기준}

---
생성 team: 01_planning-team
생성일시: {YYYY-MM-DD HH:mm}
iteration: {N} (재분석 시 증가)
```

### 4.3 result_{팀명}.md

각 팀이 수행 완료 후 결과를 기록한다.

```markdown
# Result: {팀명}

## 1. 수행 내용
{수행한 작업 요약}

## 2. 변경 파일 목록
| 파일 경로 | 변경 유형 | 설명 |
|----------|----------|------|
| {경로} | 생성/수정/삭제 | {설명} |

## 3. 주요 결정 사항
{아키텍처/설계/구현 결정 사항}

## 4. 다음 팀 참고 사항
{다음 팀이 반드시 알아야 할 내용}

## 5. 이슈/리스크
{발견된 이슈 또는 리스크}

---
수행 팀: {팀명}
참여 에이전트: {에이전트 목록}
수행일시: {YYYY-MM-DD HH:mm}
iteration: {N}
```

### 4.4 result_VERIFICATION.md

**04_qa-team**이 검증 결과를 기록한다.

```markdown
# Verification Result

## 검증 회차 기록

### Round {N}
- 검증 주체: 04_qa-team
- 검증일시: {YYYY-MM-DD HH:mm}
- 검증 기준: task_prompt.md (iteration {N}) + prompt.md
- 결과: 이상있음 / 이상없음
- 발견 사항:
  | 항목 | 상태 | 심각도 | 설명 | 대상 팀 |
  |------|------|--------|------|---------|
  | {항목} | PASS/FAIL | Critical/Major/Minor | {설명} | {수정 담당 팀} |
- 분기:
  - 이상있음 → STEP 3 (대상 팀: {팀명})
  - 이상없음 → STEP 2 (01_planning-team 재분석)

### Round {N+1}
...

## 최종 결과
- 총 반복 횟수: {N}회 / 최대 20회
- 연속 이상없음 횟수: {N}회
- 최종 판정: PASS / FAIL / ABORT (20회 초과)
- 빌드 결과: SUCCESS / FAIL
- 테스트 결과: SUCCESS / FAIL

---
검증 주체: 04_qa-team
최종 검증일시: {YYYY-MM-DD HH:mm}
```

---

## 5. Team 수행 규칙

### 5.1 각 팀의 입력

모든 팀은 수행 전 다음을 읽는다:

1. `prompt.md` (사용자 원문)
2. `task_prompt.md` (01_planning-team이 생성한 수행 프로세스)
3. 이전 팀의 `result_{팀명}.md` (해당하는 경우 — 모든 이전 result)
4. `AGENT_TEAM_STRATEGY.md`에 따른 `docs-claude/` 가이드라인 문서

### 5.2 각 팀의 출력

모든 팀은 수행 완료 후 다음을 생성한다:

1. `result_{팀명}.md` (결과 기록)
2. 실제 코드/설정 변경 (해당하는 경우)

### 5.3 팀 간 핸드오프 (순차)

```
팀 A 완료 → result_A.md 생성
    ↓
팀 B 시작 → AGENT_TEAM_STRATEGY.md에 따른 docs-claude 문서 로딩
             + prompt.md + task_prompt.md + result_A.md 읽기
    ↓
팀 B 완료 → result_B.md 생성
    ↓
팀 C 시작 → docs-claude 문서 로딩
             + prompt.md + task_prompt.md + result_A.md + result_B.md 읽기
```

### 5.4 팀 간 핸드오프 규칙

#### 전체 워크플로우 핸드오프

프롬프트 범위에 따라 아래 흐름 중 task_prompt.md에서 지정한 구간을 수행한다.

```
01_planning-team ──→ 02_design-team ──→ 03_development-team ──→ 04_qa-team
    (기획)              (설계)              (개발)                (검증)

[전달물]           [전달물]            [전달물]             [전달물]
result_01_         result_02_          result_03_           result_
planning-team.md   design-team.md      development-team.md  VERIFICATION.md
```

#### 핸드오프 전달물 규칙

| 발신 팀 | 수신 팀 | 전달물 | 수신 팀 필수 확인 사항 |
|---------|---------|--------|----------------------|
| 01_planning-team | 02_design-team | task_prompt.md + result_01 | 수행 범위, 적용 팀, 검증 기준 |
| 02_design-team | 03_development-team | result_02 | 디렉토리 구조, 컴포넌트 설계, API 인터페이스, 구현 순서 |
| 03_development-team | 04_qa-team | result_03 (+ 코드 변경) | 변경 파일 목록, 자체 검증 결과, 이슈/리스크 |
| 04_qa-team (이상있음) | 해당 팀 | result_VERIFICATION (Round N) | FAIL 항목, 심각도, 수정 가이드 |
| 04_qa-team (이상없음) | 01_planning-team | result_VERIFICATION (Round N) | 이상없음 확인, 재분석 필요 여부 |

#### 팀 건너뛰기 규칙

task_prompt.md에서 프롬프트 범위에 따라 중간 팀을 건너뛸 수 있다.

| 프롬프트 유형 | 수행 팀 | 건너뛰는 팀 |
|-------------|---------|------------|
| 신규 기능 추가 | 기획 → 설계 → 개발 → QA | 없음 (전체 수행) |
| 기존 기능 변경 | 설계 → 개발 → QA | 기획 |
| 버그 수정 | 개발 → QA | 기획, 설계 |
| 스타일/마크업 수정 | 개발 → QA | 기획, 설계 |
| 문서/설정 변경 | 해당 팀 → QA | 나머지 |

**건너뛰기 판단은 01_planning-team이 task_prompt.md에서 결정한다.**

### 5.5 팀 간 핸드오프 (병렬 포함)

전체 프로세스가 팀 A → 팀 B → 팀 C → 팀 D 단계로 이루어지고, 팀 B와 팀 C가 병렬 가능하다고 판단될 때:

```
[STEP] 팀 A 순차 수행
    │
    ▼
[STEP] 팀 B-1, B-2 병렬 수행
    │  각 팀이 독립 수행 후 산출물(result md) 각각 생성
    │  B-1 → result_{팀명}-{대상1}.md
    │  B-2 → result_{팀명}-{대상2}.md
    │
    ▼  (B-1, B-2 모두 완료될 때까지 대기)
[STEP] 팀 C 순차 수행
    │  입력: prompt.md + task_prompt.md + 이전 모든 result
```

**핵심 규칙:**

1. 순차 단계는 이전 단계 완료 후 수행한다
2. 병렬 단계의 각 팀/에이전트는 독립 수행하며, 산출물을 각각 생성한다
3. 병렬 단계의 모든 팀이 완료된 이후에만 다음 단계로 진행한다
4. 다음 단계의 입력에는 이전 병렬 단계의 모든 산출물이 포함된다

---

## 6. 병렬 처리 규칙

### 6.1 병렬 가능 조건

다음 조건을 모두 만족할 때 병렬 수행 가능:

- 두 팀/에이전트 간 입력/출력 의존성이 없음
- 서로 다른 모듈/파일을 대상으로 함
- task_prompt.md에서 "병렬 가능: Y"로 표기됨

### 6.2 병렬 실행 프로세스

| 단계 | 수행 방식 | 다음 단계 진행 조건 |
|------|----------|-------------------|
| A | 순차 수행 | A 완료 |
| B (B-1, B-2) | 병렬 수행 | B-1, B-2 모두 완료 + 산출물 생성 |
| C | 순차 수행 | - |

**병렬 단계 입력 파일:**
- 병렬 팀/에이전트는 prompt.md + task_prompt.md + 이전 단계까지의 모든 result를 읽는다
- 같은 병렬 그룹 내 다른 팀의 result는 읽지 않는다 (아직 미생성)

### 6.3 일반적인 병렬 패턴

프로젝트 워크플로우에서 병렬 수행이 가능한 대표적인 패턴이다.

#### 패턴 A: 03_development-team — 프로젝트별 병렬 개발

hs-common 변경이 완료된 후, webadmin/appuser/publishing을 병렬 개발한다.
hs-common 변경이 없으면 dev-common을 건너뛰고 즉시 병렬 개발에 진입한다.

```
[순차] dev-common (hs-common 변경 — 변경 없으면 생략)
    │
    ▼  완료 대기
[병렬] dev-webadmin ──┬── dev-appuser ──┬── dev-publishing
    │  result_03_    │  result_03_     │  result_03_
    │  development-  │  development-   │  development-
    │  team-         │  team-          │  team-
    │  webadmin.md   │  appuser.md     │  publishing.md
    │                │                 │
    ▼  모두 완료 대기 ─┴─────────────────┘
[순차] test-engineer (테스트 작성)
```

**적용 조건:**
- hs-common 변경 시: 빌드 확인이 완료된 상태에서만 병렬 진입
- hs-common 미변경 시: 즉시 병렬 개발 진입
- 각 프로젝트가 서로 다른 디렉토리에서 작업
- 동일 파일 수정 충돌 없음

#### 패턴 B: 04_qa-team — 보안/접근성/성능 병렬 검토

코드 리뷰(게이트)가 통과한 후, 보안 검토와 보조 검토를 병렬 수행한다.

```
[순차] code-reviewer (코드 리뷰 — 게이트)
    │
    ▼  통과 시에만 진행
[병렬] security-auditor ──┬── accessibility-checker ──┬── performance-analyst
    │                    │                          │
    ▼  모두 완료 대기 ────┴──────────────────────────┘
[순차] 종합 판정 → result_VERIFICATION.md
```

**적용 조건:**
- 코드 리뷰에서 Critical/Major 이슈 0건 (게이트 통과)
- 각 검토가 코드를 변경하지 않음 (읽기 전용 검토)

#### 패턴 C: 02_design-team — 아키텍처/API 병렬 설계

전체 아키텍처가 확정된 후, 컴포넌트 상세 설계와 API 상세 설계를 병렬 수행한다.

```
[순차] architect (전체 아키텍처 확정)
    │
    ▼  완료 대기
[병렬] architect (컴포넌트 상세) ──┬── api-designer (API 상세)
    │  result_02_design-         │  result_02_design-
    │  team-component.md         │  team-api.md
    │                            │
    ▼  모두 완료 대기 ────────────┘
[순차] 설계 문서 통합
```

#### 패턴 D: 01_planning-team — 플랫폼별 병렬 기획

요구사항 분석이 완료되고 양 플랫폼에 독립적인 기능인 경우:

```
[순차] requirements-analyst (요구사항 분석)
    │
    ▼  완료 대기
[병렬] planner (webadmin 기획) ──┬── planner (appuser 기획)
    │  result_01_planning-      │  result_01_planning-
    │  team-webadmin.md         │  team-appuser.md
    │                           │
    ▼  모두 완료 대기 ───────────┘
[순차] UX 리서처 (통합 UX 검증)
```

### 6.4 병렬 불가 패턴

아래 상황에서는 반드시 순차 수행해야 한다.

#### 패턴 X: hs-common → 호스트 프로젝트

```
[순차 필수] dev-common → dev-webadmin / dev-appuser
```

**불가 사유:** hs-common의 타입/유틸리티 변경이 webadmin/appuser의 import에 영향을 미친다. 반드시 hs-common 변경을 먼저 완료하고 빌드를 확인한 후 호스트 프로젝트 개발에 진입해야 한다.

#### 패턴 Y: 코드 리뷰 → 보안 검토

```
[순차 필수] code-reviewer → security-auditor
```

**불가 사유:** 코드 리뷰에서 FSD 위반이나 타입 오류가 발견되면 코드 자체가 수정되어야 한다. 코드 리뷰가 게이트 역할을 하므로 반드시 통과 후 보안 검토를 진행한다.

#### 패턴 Z: 개발 → 품질관리

```
[순차 필수] 03_development-team 전체 완료 → 04_qa-team 시작
```

**불가 사유:** 개발 진행 중인 코드를 리뷰하면 미완성 상태에서 거짓 양성(false positive) 이슈가 발생한다.

#### 패턴 W: 검증 완료 → 빌드/테스트

```
[순차 필수] 검증 PASS (연속 2회) → Build & Test
```

**불가 사유:** 검증에서 이상 발견 시 코드 수정이 발생하며, 수정된 코드로 빌드해야 한다.

#### 패턴 V: 서브모듈 커밋 순서

```
[순차 필수] hs-common 커밋 → webadmin 커밋 → appuser 커밋 → 워크스페이스 해시 반영
```

**불가 사유:** Git Submodule 구조상 하위 모듈의 커밋 해시가 확정되어야 상위에서 참조할 수 있다.

---

## 7. 검증 프로세스

### 7.1 검증 주체

**04_qa-team**이 검증을 수행한다.

참여 에이전트:
- **code-reviewer** (필수) — FSD 규칙, 코딩 컨벤션, 코드 품질
- **security-auditor** (필수) — 보안 취약점
- accessibility-checker (보조) — 접근성
- performance-analyst (보조) — 성능

### 7.2 검증 수행 조건

task_prompt.md에서 지정한 모든 팀의 수행이 완료된 후 검증을 시작한다.

### 7.3 검증 기준

1. `prompt.md`의 요구사항이 모두 구현되었는가
2. `task_prompt.md`의 수행 순서가 모두 완료되었는가
3. 각 `result_{팀명}.md`의 이슈가 모두 해결되었는가
4. `AGENT_TEAM_STRATEGY.md`에 따른 docs-claude 기준 충족
5. 코드 리뷰 체크리스트 (`05_qa/CODE_REVIEW_CHECKLIST.md`) 통과
6. 보안 체크리스트 (`02_security/SECURITY_CHECKLIST.md`) 통과

### 7.4 검증 결과 분기

```
04_qa-team 검증
    │
    ├── 이상있음 ──→ STEP 3으로 돌아감
    │                해당 팀이 수정 후 result를 갱신한다
    │                갱신 후 다시 04_qa-team이 검증한다
    │
    └── 이상없음 ──→ STEP 2로 돌아감
                     01_planning-team이 task_prompt.md를 재분석한다
                     (iteration 증가)
                     재분석 후 STEP 3 → STEP 4 다시 수행

                     이상없음이 연속 2회 도출되면 → 검증 완료 → STEP 5
```

### 7.5 검증 실패 시 재수행 규칙

| 상황 | 분기 | 재수행 범위 | result 처리 |
|------|------|-----------|-------------|
| 이상있음 | → STEP 3 | VERIFICATION에 명시된 대상 팀만 재수행 | 해당 팀의 result를 **덮어쓰기** (이전 버전은 Round 기록으로 보존) |
| 이상없음 | → STEP 2 | 01_planning-team이 task_prompt.md 재분석 → 전체 STEP 3~4 재수행 | task_prompt.md의 iteration 증가, 기존 result 유지 |

**재수행 시 입력:**
- 재수행 팀은 prompt.md + 최신 task_prompt.md + result_VERIFICATION.md (해당 Round) + docs-claude 문서를 읽는다
- result_VERIFICATION.md의 FAIL 항목과 수정 가이드를 참고하여 수정한다

### 7.6 최대 반복 횟수

**STEP 2~4의 총 반복 횟수는 최대 20회이다.**

```
카운트 대상: STEP 2 (task_prompt 생성/재분석) + STEP 4 (검증) 합산

Round  1: STEP 2 → STEP 3 → STEP 4 (이상있음)     [누적: 1회]
Round  2: STEP 3 (수정) → STEP 4 (이상없음)         [누적: 2회]
Round  3: STEP 2 (재분석) → STEP 3 → STEP 4 (이상없음)  [누적: 3회]
→ 연속 2회 이상없음 → 검증 완료

...

Round 20: STEP 4 (이상있음)                         [누적: 20회]
→ 최대 횟수 도달 → ABORT
→ result_VERIFICATION.md에 "ABORT: 최대 반복 횟수(20회) 초과" 기록
→ 사용자에게 상황 보고 및 판단 요청
```

### 7.7 검증 완료 조건 요약

| 조건 | 결과 |
|------|------|
| 이상없음 연속 2회 | **PASS** → STEP 5 진행 |
| 20회 반복 내 연속 2회 미달성 | **ABORT** → 사용자 판단 요청 |

---

## 8. Build & Test (STEP 5)

검증이 PASS(이상없음 연속 2회)된 후, **dev 모드**에서 빌드와 테스트를 수행한다.

### 8.1 수행 순서

#### STEP 5-1: 타입 체크 + 린트

변경된 프로젝트에 대해 수행한다. 하나라도 실패하면 중단하고 수정한다.

```bash
# webadmin 변경 시
cd 31_webadmin && npm run type-check && npm run lint

# appuser 변경 시
cd 32_appuser && npm run type-check && npm run lint
```

| 검사 항목 | 명령어 | 통과 기준 |
|-----------|--------|-----------|
| TypeScript 타입 체크 | `npm run type-check` | 에러 0건 |
| ESLint | `npm run lint` | 에러 0건 (경고는 허용) |

#### STEP 5-2: 단위 테스트

```bash
# webadmin (Vitest)
cd 31_webadmin && npm run test:run
```

| 검사 항목 | 명령어 | 통과 기준 |
|-----------|--------|-----------|
| Vitest 단위 테스트 | `npm run test:run` | 전체 통과 |

#### STEP 5-3: E2E 테스트

```bash
# webadmin (Playwright)
cd 31_webadmin && npm run e2e
```

| 검사 항목 | 명령어 | 통과 기준 |
|-----------|--------|-----------|
| Playwright E2E | `npm run e2e` | 전체 통과 |

#### STEP 5-4: 개발 환경 빌드

```bash
# webadmin 변경 시
cd 31_webadmin && npm run build

# appuser 변경 시
cd 32_appuser && npm run build
```

| 검사 항목 | 명령어 | 통과 기준 |
|-----------|--------|-----------|
| 개발 빌드 | `npm run build` | 빌드 성공 |

#### STEP 5-5: 결과 기록

모든 빌드/테스트 결과를 `result_VERIFICATION.md`의 최종 결과 섹션에 기록한다.

```markdown
## 빌드/테스트 결과 (dev 모드)

| 단계 | 대상 | 결과 | 비고 |
|------|------|------|------|
| 타입 체크 | webadmin | SUCCESS / FAIL | |
| 타입 체크 | appuser | SUCCESS / FAIL | |
| 린트 | webadmin | SUCCESS / FAIL | |
| 린트 | appuser | SUCCESS / FAIL | |
| 단위 테스트 | webadmin | SUCCESS / FAIL | N개 중 M개 통과 |
| E2E 테스트 | webadmin | SUCCESS / FAIL | N개 중 M개 통과 |
| 개발 빌드 | webadmin | SUCCESS / FAIL | |
| 개발 빌드 | appuser | SUCCESS / FAIL | |
```

### 8.2 실패 시 처리

| 실패 단계 | 조치 |
|-----------|------|
| 타입 체크/린트 실패 | 코드 수정 → STEP 3으로 돌아감 (해당 팀 재수행 + 재검증) |
| 단위 테스트 실패 | 테스트 또는 코드 수정 → STEP 3으로 돌아감 |
| E2E 테스트 실패 | 테스트 또는 코드 수정 → STEP 3으로 돌아감 |
| 개발 빌드 실패 | 빌드 에러 수정 → STEP 3으로 돌아감 |

**Build & Test에서 코드 수정이 발생하면 STEP 3으로 돌아가 해당 팀이 수정 후 STEP 4(04_qa-team 검증)를 다시 수행한다. 이때 7.6의 최대 반복 횟수에 포함된다.**

### 8.3 배포 프로세스

**현재 미정.** 운영 빌드(`npm run build:prod`) 및 배포 절차는 추후 정의한다.

---

## 9. 결과물 보존 정책

- `result-docs/**` 하위의 모든 파일은 삭제하지 않는다
- `result-docs/**`는 `.gitignore`에 등록하여 Git 추적에서 제외한다
- 결과물은 향후 참조 및 감사 용도로 보존한다
- 검증 Round별 기록은 `result_VERIFICATION.md`에 누적 기록되어 이력이 보존된다

---

## 10. Superpowers 플러그인 활용

Team 수행 시 각 STEP에서 적합한 superpowers 플러그인을 활용한다.

### 10.1 STEP별 플러그인 매핑

```
STEP 2. 01_planning-team → task_prompt.md 생성
    ├── /brainstorming → 기획·요구사항 탐색
    └── /writing-plans → 구현 계획 수립

STEP 3. Team 순차/병렬 수행
    ├── /frontend-design → UI 컴포넌트·페이지 생성/개선 시
    ├── /executing-plans → 계획 기반 구현 수행
    ├── /dispatching-parallel-agents → 프로젝트별 병렬 개발 (패턴 A)
    └── /test-driven-development → TDD 적용 시

STEP 4. 04_qa-team → 검증
    ├── /requesting-code-review → 코드 리뷰 요청
    ├── /receiving-code-review → 리뷰 피드백 처리
    ├── /verification-before-completion → 완료 전 최종 검증
    └── /systematic-debugging → 디버깅 필요 시

STEP 5. Build & Test 완료 후
    └── /finishing-a-development-branch → 개발 완료 후 통합
```

### 10.2 플러그인과 기존 프로세스의 관계

| 기존 프로세스 | 플러그인 | 보완 역할 |
|-------------|---------|----------|
| 01_planning-team의 task_prompt.md 생성 | `/brainstorming` + `/writing-plans` | 요구사항 탐색 강화, 계획 구조화 |
| 03_development-team UI 구현 | `/frontend-design` | 프로덕션 수준의 고품질 UI 컴포넌트·페이지 생성 |
| 03_development-team 순차 수행 | `/executing-plans` | 계획 기반 체계적 실행 |
| 03_development-team 병렬 수행 (패턴 A) | `/dispatching-parallel-agents` | webadmin/appuser/publishing 병렬 개발 |
| 04_qa-team code-reviewer 검증 | `/requesting-code-review` + `/receiving-code-review` | 리뷰 프로세스 체계화 |
| 04_qa-team 검증 완료 판정 | `/verification-before-completion` | 완료 선언 전 증거 기반 검증 |

### 10.3 적용 규칙

- `[team]` 작업 시 superpowers 플러그인 사용은 **필수**이다
- 플러그인은 기존 TEAM_EXECUTION_PROTOCOL을 **대체하지 않고 보완**한다
- 기존 result_*.md 생성, 검증 회차(최대 20회), 빌드/테스트 프로세스는 그대로 유지한다
- 04_qa-team 검증 시 `/code-review`, `/simplify`, `/security-guidance` 플러그인도 함께 활용한다

---

END OF FILE
