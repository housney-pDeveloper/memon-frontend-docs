# AGENT_DOCS_STRATEGY.md

에이전트별 참조해야 할 가이드라인 문서를 정의합니다.
각 에이전트는 업무 수행 시 연결된 문서를 **필수 참조**하여 프로젝트 일관성을 유지합니다.

---

## 문서 경로 기준

- 에이전트: `docs-agents/agent/`
- 가이드라인: `docs-claude/`

---

## 1. 필수 에이전트

### 01_requirements-analyst (요구사항 분석)

> 요구사항의 기술적 실현 가능성과 플랫폼 영향도를 판단하기 위해 아키텍처와 데이터 흐름을 이해해야 한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `01_architecture/FSD_ARCHITECTURE.md` | 기능 배치 가능 레이어 판단, FSD 제약 조건 파악 |
| **필수** | `01_architecture/STATE_MANAGEMENT.md` | 상태 관리 전략이 요구사항에 미치는 영향 평가 |
| **필수** | `03_data/API_CONVENTION.md` | 백엔드 API 연동 방식 이해, Gateway 패턴 확인 |
| 참고 | `03_data/ERROR_HANDLING.md` | 에러 코드 체계 이해, 에러 처리 요구사항 도출 |
| 참고 | `04_dev-app/PLATFORM_GUIDE.md` | appuser v2(RN) 전환 호환성 판단 |

---

### 02_planner (기획)

> 화면 흐름, 상태 관리 전략, 라우팅 구조를 이해해야 구체적인 기능 명세를 작성할 수 있다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `01_architecture/ROUTING_GUIDE.md` | 페이지 구조, 가드 체계, 미들웨어 이해 |
| **필수** | `01_architecture/STATE_MANAGEMENT.md` | 상태 유형별 관리 도구 결정 |
| **필수** | `03_data/API_CONVENTION.md` | API 엔드포인트 초안 작성 시 패턴 준수 |
| 참고 | `01_architecture/FSD_ARCHITECTURE.md` | 기능 명세의 FSD 레이어 배치 방향 |
| 참고 | `02_security/AUTH_GUIDE.md` | 인증/권한 관련 기능 기획 시 참조 |

---

### 03_architect (설계)

> 아키텍처 전반을 설계하므로 모든 아키텍처/데이터 문서를 숙지해야 한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `01_architecture/FSD_ARCHITECTURE.md` | 레이어 배치, 의존성 규칙, 컴포넌트 분류 기준 |
| **필수** | `01_architecture/STATE_MANAGEMENT.md` | 쿼리 키 설계, 스토어 구조 설계 |
| **필수** | `01_architecture/ROUTING_GUIDE.md` | 라우트 구조 및 가드 설계 |
| **필수** | `03_data/API_CONVENTION.md` | API 배치 규칙 (entities/GET vs features/변경) |
| **필수** | `03_data/TYPE_CONVENTION.md` | 타입 명명/배치 규칙 준수 |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 처리 구조 설계 |
| 참고 | `02_security/AUTH_GUIDE.md` | 인증 아키텍처 이해 (콜백 패턴 등) |
| 참고 | `04_dev-web/COMPONENT_GUIDE.md` | UI 계층 구조 참조 |

---

### 04_dev-webadmin (웹 관리자 개발)

> webadmin 코드를 직접 작성하므로 해당 프로젝트의 모든 컨벤션을 준수해야 한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `04_dev-web/CODE_CONVENTION.md` | 명명 규칙, 컴포넌트 작성, import 순서, 금지 사항 |
| **필수** | `04_dev-web/COMPONENT_GUIDE.md` | UI 계층, cva 패턴, 공통 컴포넌트 사용법 |
| **필수** | `01_architecture/FSD_ARCHITECTURE.md` | Full FSD 레이어 규칙 준수 |
| **필수** | `03_data/API_CONVENTION.md` | API 함수 작성 패턴, buildApiUrl 사용 |
| **필수** | `03_data/TYPE_CONVENTION.md` | 타입 정의 규칙 |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 처리 패턴 (getApiErrorMessage) |
| 참고 | `04_dev-web/TESTING_GUIDE.md` | 테스트 작성 시 참조 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | 상태 관리 패턴 |
| 참고 | `02_security/AUTH_GUIDE.md` | 인증 관련 기능 구현 시 참조 |
| 참고 | `GIT_POLICY.md` | 커밋 메시지 규칙 |

---

### 04_dev-appuser (앱 사용자 개발)

> appuser의 고유 제약(Radix 금지, core/ 필수)과 v2 전환을 항상 고려해야 한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `04_dev-app/CODE_CONVENTION.md` | Radix UI 금지, core/ 사용 필수, 순수 Tailwind |
| **필수** | `04_dev-app/PLATFORM_GUIDE.md` | IStorage 인터페이스, v2 전환 대비, 모바일 UX |
| **필수** | `01_architecture/FSD_ARCHITECTURE.md` | Light FSD 레이어 규칙 (core/ 포함) |
| **필수** | `03_data/API_CONVENTION.md` | API 함수 패턴 (clientType: 'app') |
| **필수** | `03_data/TYPE_CONVENTION.md` | 타입 정의 규칙 |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 처리 패턴 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | 상태 관리 패턴 |
| 참고 | `02_security/AUTH_GUIDE.md` | 인증 흐름 (core/ 스토리지 연동) |
| 참고 | `GIT_POLICY.md` | 커밋 메시지 규칙 |

---

### 04_dev-common (공통 라이브러리 개발)

> hs-common은 양쪽 프로젝트에 영향을 미치므로 타입/API 컨벤션을 엄격히 준수해야 한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `03_data/TYPE_CONVENTION.md` | 공유 타입 정의 규칙 (ServerResponse, ApiError 등) |
| **필수** | `03_data/API_CONVENTION.md` | API 유틸리티 설계 (createApiConfig, buildApiUrl) |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 유틸리티 설계 (parseApiError, getApiErrorMessage) |
| 참고 | `04_dev-web/CODE_CONVENTION.md` | webadmin에서의 사용 패턴 확인 |
| 참고 | `04_dev-app/CODE_CONVENTION.md` | appuser에서의 사용 패턴 확인 |

---

### 04_dev-publishing (퍼블리싱 개발)

> 퍼블리싱 고유 규칙을 따르며, 비즈니스 로직을 포함하지 않는다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `04_dev-pub/CODE_CONVENTION.md` | 퍼블리싱 규칙 (참조용, 로직 금지) |
| **필수** | `04_dev-pub/MARKUP_GUIDE.md` | Tailwind, 디자인 토큰, 반응형, cva 패턴 |
| 참고 | `04_dev-web/COMPONENT_GUIDE.md` | webadmin UI 계층 참조 (퍼블리싱 결과물 연계) |
| 참고 | `05_qa/ACCESSIBILITY_GUIDE.md` | 시맨틱 HTML, ARIA 기본 적용 |

---

### 05_code-reviewer (코드 리뷰)

> 모든 프로젝트의 코드를 검증하므로 아키텍처/컨벤션/보안 문서를 폭넓게 참조한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `05_qa/CODE_REVIEW_CHECKLIST.md` | 리뷰 체크리스트 (9개 섹션) 기준 |
| **필수** | `01_architecture/FSD_ARCHITECTURE.md` | FSD 의존성 규칙 위반 탐지 |
| **필수** | `04_dev-web/CODE_CONVENTION.md` | webadmin 코딩 컨벤션 검증 |
| **필수** | `04_dev-app/CODE_CONVENTION.md` | appuser 코딩 컨벤션 검증 |
| **필수** | `03_data/API_CONVENTION.md` | API 배치/작성 규칙 검증 |
| **필수** | `03_data/TYPE_CONVENTION.md` | 타입 규칙 검증 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | 상태 관리 패턴 검증 |
| 참고 | `02_security/SECURITY_CHECKLIST.md` | 보안 이슈 사전 탐지 |
| 참고 | `03_data/ERROR_HANDLING.md` | 에러 처리 패턴 검증 |

---

### 06_security-auditor (보안 검토)

> 보안 문서를 기준으로 코드의 보안 취약점을 감사한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `02_security/AUTH_GUIDE.md` | 인증/인가 플로우, 토큰 관리 검증 |
| **필수** | `02_security/SECURITY_CHECKLIST.md` | 보안 체크리스트 기준 감사 |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 메시지 노출, 자동 리다이렉트 검증 |
| 참고 | `01_architecture/FSD_ARCHITECTURE.md` | 콜백 패턴 보안성 검증 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | Zustand persist 보안 검증 |
| 참고 | `04_dev-app/PLATFORM_GUIDE.md` | 모바일 스토리지 보안 검증 |

---

### 07_build-deployer (빌드/배포)

> 빌드 검증 및 커밋 규칙을 준수하여 배포한다.

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `GIT_POLICY.md` | 커밋 메시지 규칙, 서브모듈 커밋 순서 |
| **필수** | `05_qa/TESTING_STANDARD.md` | 품질 게이트 (type-check, lint, test, e2e) |
| 참고 | `02_security/SECURITY_CHECKLIST.md` | 배포 전 보안 점검 (npm audit, 소스맵) |
| 참고 | `03_data/API_CONVENTION.md` | 환경 변수 설정 확인 |

---

## 2. 보조 에이전트

### sub_ux-researcher (UX 리서처)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `05_qa/ACCESSIBILITY_GUIDE.md` | 접근성 초기 요구사항 도출 |
| **필수** | `04_dev-app/PLATFORM_GUIDE.md` | 모바일 UX 가이드라인, 터치 타겟 |
| 참고 | `04_dev-web/COMPONENT_GUIDE.md` | 데스크톱 UI 패턴 이해 |
| 참고 | `04_dev-pub/MARKUP_GUIDE.md` | 반응형 디자인 브레이크포인트 |

---

### sub_api-designer (API 설계 보조)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `03_data/API_CONVENTION.md` | Gateway 패턴, API 함수 작성 규칙 |
| **필수** | `03_data/TYPE_CONVENTION.md` | 요청/응답 타입 명명 및 정의 규칙 |
| **필수** | `03_data/ERROR_HANDLING.md` | 에러 코드 체계, 에러 시나리오 정의 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | TanStack Query 쿼리 키 설계 |

---

### sub_test-engineer (테스트 엔지니어)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `04_dev-web/TESTING_GUIDE.md` | Vitest/Playwright 작성 패턴 |
| **필수** | `05_qa/TESTING_STANDARD.md` | 커버리지 목표, 품질 게이트, 목업 전략 |
| 참고 | `04_dev-web/CODE_CONVENTION.md` | 테스트 코드도 컨벤션 준수 |
| 참고 | `01_architecture/STATE_MANAGEMENT.md` | 상태 관리 목업 방법 이해 |

---

### sub_accessibility-checker (접근성 검토)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `05_qa/ACCESSIBILITY_GUIDE.md` | WCAG 2.1 AA 기준, ARIA, 키보드 접근성 |
| 참고 | `04_dev-web/COMPONENT_GUIDE.md` | 컴포넌트별 접근성 적용 방법 |
| 참고 | `04_dev-app/PLATFORM_GUIDE.md` | 모바일 접근성 (터치 타겟, 줌) |
| 참고 | `04_dev-pub/MARKUP_GUIDE.md` | 시맨틱 HTML 구조 |

---

### sub_performance-analyst (성능 분석)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `01_architecture/STATE_MANAGEMENT.md` | Zustand selector, 쿼리 캐시 전략 |
| **필수** | `04_dev-web/COMPONENT_GUIDE.md` | 컴포넌트 렌더링 최적화 |
| 참고 | `01_architecture/ROUTING_GUIDE.md` | 코드 스플리팅, 라우트별 번들 |
| 참고 | `04_dev-app/PLATFORM_GUIDE.md` | 모바일 성능 기준 (3G, 터치 반응) |

---

### sub_release-manager (릴리스 관리)

| 참조 수준 | 문서 | 참조 목적 |
|-----------|------|-----------|
| **필수** | `GIT_POLICY.md` | 커밋 규칙, 서브모듈 관리 |
| **필수** | `05_qa/TESTING_STANDARD.md` | 품질 게이트 통과 확인 |
| 참고 | `02_security/SECURITY_CHECKLIST.md` | 배포 전 보안 확인 (npm audit) |

---

## 3. 문서별 역참조 (어떤 에이전트가 이 문서를 참조하는가)

| 문서 | 필수 참조 에이전트 | 참고 참조 에이전트 |
|------|-------------------|-------------------|
| `GIT_POLICY.md` | build-deployer, release-manager | dev-webadmin, dev-appuser |
| `01_architecture/FSD_ARCHITECTURE.md` | requirements-analyst, architect, dev-webadmin, dev-appuser, code-reviewer | security-auditor |
| `01_architecture/STATE_MANAGEMENT.md` | requirements-analyst, architect | dev-webadmin, dev-appuser, code-reviewer, api-designer, test-engineer, performance-analyst, security-auditor |
| `01_architecture/ROUTING_GUIDE.md` | planner, architect | performance-analyst |
| `02_security/AUTH_GUIDE.md` | security-auditor | planner, dev-webadmin, dev-appuser, architect |
| `02_security/SECURITY_CHECKLIST.md` | security-auditor | code-reviewer, build-deployer, release-manager |
| `03_data/API_CONVENTION.md` | requirements-analyst, planner, architect, dev-webadmin, dev-appuser, dev-common, code-reviewer, api-designer | build-deployer |
| `03_data/TYPE_CONVENTION.md` | architect, dev-webadmin, dev-appuser, dev-common, code-reviewer, api-designer | — |
| `03_data/ERROR_HANDLING.md` | dev-webadmin, dev-appuser, dev-common, security-auditor, api-designer | requirements-analyst, code-reviewer |
| `04_dev-web/CODE_CONVENTION.md` | dev-webadmin, code-reviewer | dev-common, test-engineer |
| `04_dev-web/COMPONENT_GUIDE.md` | dev-webadmin | architect, ux-researcher, accessibility-checker, performance-analyst |
| `04_dev-web/TESTING_GUIDE.md` | test-engineer | dev-webadmin |
| `04_dev-app/CODE_CONVENTION.md` | dev-appuser, code-reviewer | dev-common |
| `04_dev-app/PLATFORM_GUIDE.md` | dev-appuser, ux-researcher | requirements-analyst, security-auditor, accessibility-checker, performance-analyst |
| `04_dev-pub/CODE_CONVENTION.md` | dev-publishing | — |
| `04_dev-pub/MARKUP_GUIDE.md` | dev-publishing | ux-researcher, accessibility-checker |
| `05_qa/CODE_REVIEW_CHECKLIST.md` | code-reviewer | — |
| `05_qa/TESTING_STANDARD.md` | test-engineer, build-deployer, release-manager | — |
| `05_qa/ACCESSIBILITY_GUIDE.md` | ux-researcher, accessibility-checker | dev-publishing |
