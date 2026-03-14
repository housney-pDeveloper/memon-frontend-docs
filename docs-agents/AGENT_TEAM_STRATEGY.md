# AGENT_TEAM_STRATEGY.md

팀별 참조해야 할 가이드라인 문서를 정의합니다.
각 팀은 워크플로우 단계 수행 시 연결된 문서를 기준으로 일관된 품질을 유지합니다.

---

## 문서 경로 기준

- 팀: `docs-agents/team/`
- 가이드라인: `docs-claude/`

---

## 1. 기획팀 (Planning Team)

**워크플로우**: 요구사항 분석 → 기획

### 팀 문서 매핑

```
기획팀
├── [필수] 01_architecture/FSD_ARCHITECTURE.md
├── [필수] 01_architecture/STATE_MANAGEMENT.md
├── [필수] 01_architecture/ROUTING_GUIDE.md
├── [필수] 03_data/API_CONVENTION.md
├── [참고] 02_security/AUTH_GUIDE.md
├── [참고] 03_data/ERROR_HANDLING.md
├── [참고] 04_dev-app/PLATFORM_GUIDE.md
└── [참고] 05_qa/ACCESSIBILITY_GUIDE.md
```

### 에이전트별 문서 참조

| 에이전트 | 필수 문서 | 참고 문서 |
|----------|-----------|-----------|
| **요구사항 분석** | FSD_ARCHITECTURE, STATE_MANAGEMENT, API_CONVENTION | ERROR_HANDLING, PLATFORM_GUIDE |
| **기획** | ROUTING_GUIDE, STATE_MANAGEMENT, API_CONVENTION | FSD_ARCHITECTURE, AUTH_GUIDE |
| UX 리서처 (보조) | ACCESSIBILITY_GUIDE, PLATFORM_GUIDE | COMPONENT_GUIDE, MARKUP_GUIDE |

### 활용 시나리오

| 기획 활동 | 참조 문서 | 활용 방법 |
|-----------|-----------|-----------|
| 기능 배치 판단 | FSD_ARCHITECTURE | entities(읽기) vs features(쓰기) 분류 |
| 화면 흐름 설계 | ROUTING_GUIDE | 라우트 구조, 가드 적용 범위 결정 |
| 상태 관리 계획 | STATE_MANAGEMENT | 서버/클라이언트/폼/URL 상태 분류 |
| API 명세 초안 | API_CONVENTION | Gateway 패턴 URL, HTTP 메서드 결정 |
| 인증 기능 기획 | AUTH_GUIDE | 토큰 전략, 가드 계층 이해 |
| 앱 기능 기획 | PLATFORM_GUIDE | v2(RN) 호환성, core/ 추상화 고려 |
| 접근성 요구사항 | ACCESSIBILITY_GUIDE | WCAG 기준 초기 요구사항 도출 |

---

## 2. 설계팀 (Design & Architecture Team)

**워크플로우**: 설계

### 팀 문서 매핑

```
설계팀
├── [필수] 01_architecture/FSD_ARCHITECTURE.md
├── [필수] 01_architecture/STATE_MANAGEMENT.md
├── [필수] 01_architecture/ROUTING_GUIDE.md
├── [필수] 03_data/API_CONVENTION.md
├── [필수] 03_data/TYPE_CONVENTION.md
├── [필수] 03_data/ERROR_HANDLING.md
├── [참고] 02_security/AUTH_GUIDE.md
└── [참고] 04_dev-web/COMPONENT_GUIDE.md
```

### 에이전트별 문서 참조

| 에이전트 | 필수 문서 | 참고 문서 |
|----------|-----------|-----------|
| **설계** | FSD_ARCHITECTURE, STATE_MANAGEMENT, ROUTING_GUIDE, API_CONVENTION, TYPE_CONVENTION, ERROR_HANDLING | AUTH_GUIDE, COMPONENT_GUIDE |
| API 설계 보조 | API_CONVENTION, TYPE_CONVENTION, ERROR_HANDLING | STATE_MANAGEMENT |

### 활용 시나리오

| 설계 활동 | 참조 문서 | 활용 방법 |
|-----------|-----------|-----------|
| 레이어 배치 | FSD_ARCHITECTURE | 도메인별 entities/features 구조 설계 |
| 컴포넌트 설계 | FSD_ARCHITECTURE, COMPONENT_GUIDE | UI 계층 분류 (shared/ui → entities/ui → features/ui) |
| API 인터페이스 | API_CONVENTION, TYPE_CONVENTION | 엔드포인트 설계, 요청/응답 타입 정의 |
| 쿼리 키 설계 | STATE_MANAGEMENT | 쿼리 키 구조, 무효화 전략 |
| 스토어 설계 | STATE_MANAGEMENT | Zustand 인터페이스, selector 설계 |
| 에러 처리 설계 | ERROR_HANDLING | 에러 코드 매핑, 자동/수동 처리 구분 |
| 라우트 설계 | ROUTING_GUIDE | 페이지 구조, 가드 적용 범위 |

---

## 3. 개발팀 (Development Team)

**워크플로우**: 개발

### 팀 문서 매핑

```
개발팀
├── [필수] 01_architecture/FSD_ARCHITECTURE.md
├── [필수] 03_data/API_CONVENTION.md
├── [필수] 03_data/TYPE_CONVENTION.md
├── [필수] 03_data/ERROR_HANDLING.md
├── [필수] 04_dev-web/CODE_CONVENTION.md        ← webadmin 전용
├── [필수] 04_dev-web/COMPONENT_GUIDE.md        ← webadmin 전용
├── [필수] 04_dev-app/CODE_CONVENTION.md        ← appuser 전용
├── [필수] 04_dev-app/PLATFORM_GUIDE.md         ← appuser 전용
├── [필수] 04_dev-pub/CODE_CONVENTION.md        ← publishing 전용
├── [필수] 04_dev-pub/MARKUP_GUIDE.md           ← publishing 전용
├── [필수] GIT_POLICY.md
├── [참고] 01_architecture/STATE_MANAGEMENT.md
├── [참고] 02_security/AUTH_GUIDE.md
├── [참고] 04_dev-web/TESTING_GUIDE.md
└── [참고] 05_qa/TESTING_STANDARD.md
```

### 에이전트별 문서 참조

| 에이전트 | 필수 문서 | 참고 문서 |
|----------|-----------|-----------|
| **webadmin 개발** | CODE_CONVENTION(web), COMPONENT_GUIDE, FSD_ARCHITECTURE, API_CONVENTION, TYPE_CONVENTION, ERROR_HANDLING | TESTING_GUIDE, STATE_MANAGEMENT, AUTH_GUIDE, GIT_POLICY |
| **appuser 개발** | CODE_CONVENTION(app), PLATFORM_GUIDE, FSD_ARCHITECTURE, API_CONVENTION, TYPE_CONVENTION, ERROR_HANDLING | STATE_MANAGEMENT, AUTH_GUIDE, GIT_POLICY |
| **common 개발** | TYPE_CONVENTION, API_CONVENTION, ERROR_HANDLING | CODE_CONVENTION(web), CODE_CONVENTION(app) |
| **publishing 개발** | CODE_CONVENTION(pub), MARKUP_GUIDE | COMPONENT_GUIDE, ACCESSIBILITY_GUIDE |
| 테스트 엔지니어 (보조) | TESTING_GUIDE, TESTING_STANDARD | CODE_CONVENTION(web), STATE_MANAGEMENT |

### 프로젝트별 필수 문서 매트릭스

| 문서 | webadmin | appuser | common | publishing |
|------|----------|---------|--------|------------|
| `FSD_ARCHITECTURE` | **Full FSD** | **Light FSD** | 참고 | - |
| `API_CONVENTION` | **필수** | **필수** | **필수** | - |
| `TYPE_CONVENTION` | **필수** | **필수** | **필수** | - |
| `ERROR_HANDLING` | **필수** | **필수** | **필수** | - |
| `CODE_CONVENTION(web)` | **필수** | - | 참고 | - |
| `CODE_CONVENTION(app)` | - | **필수** | 참고 | - |
| `CODE_CONVENTION(pub)` | - | - | - | **필수** |
| `COMPONENT_GUIDE` | **필수** | - | - | 참고 |
| `PLATFORM_GUIDE` | - | **필수** | - | - |
| `MARKUP_GUIDE` | - | - | - | **필수** |
| `TESTING_GUIDE` | 참고 | - | - | - |
| `AUTH_GUIDE` | 참고 | 참고 | - | - |
| `GIT_POLICY` | 참고 | 참고 | 참고 | - |

---

## 4. 품질관리팀 (QA Team)

**워크플로우**: 리뷰 → 보안검토

### 팀 문서 매핑

```
품질관리팀
├── [필수] 05_qa/CODE_REVIEW_CHECKLIST.md
├── [필수] 05_qa/TESTING_STANDARD.md
├── [필수] 05_qa/ACCESSIBILITY_GUIDE.md
├── [필수] 02_security/AUTH_GUIDE.md
├── [필수] 02_security/SECURITY_CHECKLIST.md
├── [필수] 01_architecture/FSD_ARCHITECTURE.md
├── [필수] 04_dev-web/CODE_CONVENTION.md
├── [필수] 04_dev-app/CODE_CONVENTION.md
├── [필수] 03_data/API_CONVENTION.md
├── [필수] 03_data/TYPE_CONVENTION.md
├── [참고] 01_architecture/STATE_MANAGEMENT.md
├── [참고] 03_data/ERROR_HANDLING.md
├── [참고] 04_dev-web/COMPONENT_GUIDE.md
└── [참고] 04_dev-app/PLATFORM_GUIDE.md
```

### 에이전트별 문서 참조

| 에이전트 | 필수 문서 | 참고 문서 |
|----------|-----------|-----------|
| **코드 리뷰** | CODE_REVIEW_CHECKLIST, FSD_ARCHITECTURE, CODE_CONVENTION(web), CODE_CONVENTION(app), API_CONVENTION, TYPE_CONVENTION | STATE_MANAGEMENT, SECURITY_CHECKLIST, ERROR_HANDLING |
| **보안 검토** | AUTH_GUIDE, SECURITY_CHECKLIST, ERROR_HANDLING | FSD_ARCHITECTURE, STATE_MANAGEMENT, PLATFORM_GUIDE |
| 접근성 검토 (보조) | ACCESSIBILITY_GUIDE | COMPONENT_GUIDE, PLATFORM_GUIDE, MARKUP_GUIDE |
| 성능 분석 (보조) | STATE_MANAGEMENT, COMPONENT_GUIDE | ROUTING_GUIDE, PLATFORM_GUIDE |

### 검토 단계별 문서 매핑

```
1단계: 코드 리뷰 (필수)
  ├── CODE_REVIEW_CHECKLIST.md  ← 체크리스트 기준
  ├── FSD_ARCHITECTURE.md       ← 레이어 의존성 검증
  ├── CODE_CONVENTION(web/app)  ← 컨벤션 검증
  ├── API_CONVENTION.md         ← API 규칙 검증
  └── TYPE_CONVENTION.md        ← 타입 규칙 검증
       │
       ▼
2단계: 보안 검토 + 보조 검토 (병렬)
  ├── AUTH_GUIDE.md             ← 인증 보안 검증
  ├── SECURITY_CHECKLIST.md     ← 보안 체크리스트 감사
  ├── ACCESSIBILITY_GUIDE.md    ← 접근성 검증 (보조)
  └── STATE_MANAGEMENT.md       ← 성능 검증 (보조)
       │
       ▼
3단계: 종합 판정
  └── 통과 기준: 각 문서의 필수 항목 위반 0건
```

---

## 5. 배포팀 (DevOps Team)

**워크플로우**: 빌드, 배포

### 팀 문서 매핑

```
배포팀
├── [필수] GIT_POLICY.md
├── [필수] 05_qa/TESTING_STANDARD.md
├── [참고] 02_security/SECURITY_CHECKLIST.md
├── [참고] 03_data/API_CONVENTION.md
└── [참고] 01_architecture/FSD_ARCHITECTURE.md
```

### 에이전트별 문서 참조

| 에이전트 | 필수 문서 | 참고 문서 |
|----------|-----------|-----------|
| **빌드/배포** | GIT_POLICY, TESTING_STANDARD | SECURITY_CHECKLIST, API_CONVENTION |
| 릴리스 관리 (보조) | GIT_POLICY, TESTING_STANDARD | SECURITY_CHECKLIST |

### 배포 단계별 문서 매핑

```
1단계: 사전 검증
  └── TESTING_STANDARD.md       ← 품질 게이트 (type-check, lint, test, e2e)

2단계: 빌드
  └── API_CONVENTION.md         ← 환경 변수 설정 확인

3단계: 서브모듈 커밋
  └── GIT_POLICY.md             ← 커밋 메시지 규칙, 서브모듈 순서

4단계: 배포 전 점검
  └── SECURITY_CHECKLIST.md     ← npm audit, 소스맵, 민감 정보 확인

5단계: 배포 후 검증
  └── TESTING_STANDARD.md       ← 스모크 테스트 기준
```

---

## 6. 팀 간 문서 공유 매트릭스

각 문서가 어느 팀에서 어떤 수준으로 참조되는지 한눈에 파악합니다.

| 문서 | 기획팀 | 설계팀 | 개발팀 | 품질관리팀 | 배포팀 |
|------|--------|--------|--------|-----------|--------|
| `GIT_POLICY` | - | - | 참고 | - | **필수** |
| `01_architecture/FSD_ARCHITECTURE` | **필수** | **필수** | **필수** | **필수** | 참고 |
| `01_architecture/STATE_MANAGEMENT` | **필수** | **필수** | 참고 | 참고 | - |
| `01_architecture/ROUTING_GUIDE` | **필수** | **필수** | - | - | - |
| `02_security/AUTH_GUIDE` | 참고 | 참고 | 참고 | **필수** | - |
| `02_security/SECURITY_CHECKLIST` | - | - | - | **필수** | 참고 |
| `03_data/API_CONVENTION` | **필수** | **필수** | **필수** | **필수** | 참고 |
| `03_data/TYPE_CONVENTION` | - | **필수** | **필수** | **필수** | - |
| `03_data/ERROR_HANDLING` | 참고 | **필수** | **필수** | 참고 | - |
| `04_dev-web/CODE_CONVENTION` | - | - | **필수** | **필수** | - |
| `04_dev-web/COMPONENT_GUIDE` | - | 참고 | **필수** | 참고 | - |
| `04_dev-web/TESTING_GUIDE` | - | - | 참고 | - | - |
| `04_dev-app/CODE_CONVENTION` | - | - | **필수** | **필수** | - |
| `04_dev-app/PLATFORM_GUIDE` | 참고 | - | **필수** | 참고 | - |
| `04_dev-pub/CODE_CONVENTION` | - | - | **필수** | - | - |
| `04_dev-pub/MARKUP_GUIDE` | - | - | **필수** | - | - |
| `05_qa/CODE_REVIEW_CHECKLIST` | - | - | - | **필수** | - |
| `05_qa/TESTING_STANDARD` | - | - | 참고 | **필수** | **필수** |
| `05_qa/ACCESSIBILITY_GUIDE` | 참고 | - | - | **필수** | - |

---

## 7. 워크플로우 전체 흐름과 문서 연결

```
요구사항 분석 ──→ 기획 ──→ 설계 ──→ 개발 ──→ 리뷰 ──→ 보안검토 ──→ 빌드,배포
  [기획팀]      [기획팀]   [설계팀]   [개발팀]   [품질관리팀]           [배포팀]

  참조 문서:    참조 문서:  참조 문서:  참조 문서:   참조 문서:           참조 문서:
  ┌──────┐    ┌──────┐   ┌──────┐   ┌──────┐    ┌──────┐            ┌──────┐
  │01_arch│    │01_arch│  │01_arch│  │04_dev*│   │05_qa │            │GIT   │
  │03_data│    │03_data│  │03_data│  │01_arch│   │02_sec│            │05_qa │
  │      │    │02_sec │  │02_sec │  │03_data│   │01_arch│           │02_sec│
  │      │    │      │  │04_dev │  │GIT   │   │04_dev*│           │03_data│
  └──────┘    └──────┘   └──────┘   └──────┘    └──────┘            └──────┘
```

각 팀은 이전 팀의 산출물 + 해당 단계 문서를 참조하여 작업하며, 다음 팀에 산출물을 전달합니다.
