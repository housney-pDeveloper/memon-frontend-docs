# 프론트엔드 보안 체크리스트

> webadmin(관리자 대시보드)과 appuser(모바일 사용자 앱) 개발 시 준수해야 할 보안 체크리스트.
> 코드 리뷰, PR 머지 전, 배포 전에 이 항목들을 확인한다.

---

## 1. XSS (Cross-Site Scripting) 방지

### 필수 준수 사항

- [ ] **`dangerouslySetInnerHTML` 사용 금지**
  - React의 자동 이스케이핑을 우회하는 가장 위험한 패턴이다
  - 불가피한 경우 반드시 DOMPurify 등으로 sanitize 후 사용
  - 사용 시 코드 리뷰에서 반드시 보안 검토 수행

- [ ] **사용자 입력 → DOM 직접 삽입 금지**
  - `document.innerHTML`, `document.write()`, `insertAdjacentHTML()` 등 사용 금지
  - 반드시 React의 JSX를 통해 렌더링한다

- [ ] **URL 파라미터 sanitize**
  - 쿼리 파라미터, 경로 파라미터를 DOM에 표시하거나 리다이렉트에 사용할 때 검증 필수
  - `shared/lib/safeUrl.ts` 유틸리티를 사용하여 URL 안전성 확인
  - `javascript:`, `data:` 등 위험한 프로토콜 차단

- [ ] **React 자동 이스케이핑 신뢰**
  - JSX 내 `{}` 표현식은 자동으로 HTML 이스케이핑된다
  - 이 메커니즘을 우회하는 코드를 작성하지 않는다

### 확인 방법

```
코드베이스에서 다음을 검색하여 위험 패턴을 확인:
- dangerouslySetInnerHTML
- innerHTML
- document.write
- insertAdjacentHTML
- outerHTML
```

---

## 2. 민감 데이터 보호

### 필수 준수 사항

- [ ] **콘솔 로그에 민감 정보 출력 금지**
  - `console.log(token)`, `console.log(password)`, `console.log(user)` 등 금지
  - 디버깅 후 반드시 관련 로그 제거
  - 운영 빌드에서는 콘솔 로그 최소화

- [ ] **에러 메시지에 내부 시스템 정보 노출 금지**
  - API 엔드포인트 전체 경로, 서버 버전, 스택 트레이스 등을 사용자에게 표시하지 않는다
  - 사용자에게는 일반적인 에러 메시지만 표시: "요청 처리 중 오류가 발생했습니다"
  - 상세 에러는 개발자 도구 또는 로깅 시스템에서만 확인

- [ ] **`.env` 파일에 시크릿 키 직접 포함 금지**
  - `NEXT_PUBLIC_*` 접두사가 붙은 환경 변수는 클라이언트 번들에 포함된다
  - API 시크릿, 암호화 키 등은 절대 `NEXT_PUBLIC_` 접두사를 사용하지 않는다
  - 클라이언트에 노출되어도 안전한 값만 `NEXT_PUBLIC_`으로 설정

- [ ] **운영 빌드에 소스맵 비포함**
  - `next.config.ts`에서 운영 빌드 시 소스맵 생성을 비활성화
  - 소스맵이 포함되면 원본 소스코드가 브라우저 개발자 도구에서 확인 가능

- [ ] **Git에 민감 정보 커밋 금지**
  - `.env`, `.env.local`, `.env.production` 등이 `.gitignore`에 포함되어 있는지 확인
  - 실수로 커밋된 경우 이력에서도 제거 필요 (git filter-branch 또는 BFG)
  - `.env.example` 파일에는 실제 값 대신 placeholder만 기입

---

## 3. API 통신 보안

### 필수 준수 사항

- [ ] **운영 환경 HTTPS 강제**
  - 모든 API 통신은 HTTPS를 통해 이루어져야 한다
  - HTTP 요청은 서버에서 HTTPS로 리다이렉트되도록 설정

- [ ] **인증 헤더 자동 주입**
  - `Authorization: Bearer {token}` 헤더는 axios 인터셉터에서 자동 주입
  - 개별 API 호출에서 수동으로 토큰을 헤더에 추가하지 않는다
  - 인터셉터가 `getAccessToken()`으로 메모리에서 토큰을 가져온다

- [ ] **CORS 처리: Next.js Rewrites 프록시**
  - 브라우저에서 직접 외부 API를 호출하지 않는다
  - `next.config.ts`의 `rewrites`를 사용하여 `/api/*` 경로를 `API_BASE_URL`로 프록시
  - 이를 통해 CORS 문제를 서버 사이드에서 해결

- [ ] **인터셉터에서 민감 데이터 로깅 금지**
  - 요청 인터셉터: Authorization 헤더 값 로깅 금지
  - 응답 인터셉터: 토큰, 사용자 정보 등 로깅 금지
  - 디버깅 시에도 URL과 상태 코드만 로깅

- [ ] **요청 Timeout 설정**
  - `axios.create` 시 `timeout: 30000` (30초) 설정
  - 무한 대기 방지
  - 필요 시 특정 API(파일 업로드 등)에 대해 개별 timeout 조정

---

## 4. 입력 검증

### 필수 준수 사항

- [ ] **모든 폼: Zod 스키마로 클라이언트 사전 검증**
  - 서버에 요청을 보내기 전에 클라이언트에서 입력값을 검증한다
  - Zod 스키마를 정의하여 타입 안전한 검증 수행
  - react-hook-form + zodResolver 조합 사용

- [ ] **서버 검증에만 의존 금지 (이중 검증)**
  - 클라이언트 검증은 UX 개선 목적 (빠른 피드백)
  - 서버 검증은 보안 목적 (우회 방지)
  - 양쪽 모두에서 검증해야 한다

- [ ] **파일 업로드: 타입/크기 제한**
  - 허용된 MIME 타입만 업로드 가능하도록 제한
  - 파일 크기 상한 설정 (서버와 동일하게)
  - 파일명에 포함된 특수문자 처리

- [ ] **숫자 입력: `input-number.ts` 유틸리티 사용**
  - 숫자 전용 입력 필드에서 비숫자 문자 입력 방지
  - `shared/lib/input-number.ts` 또는 해당 유틸리티 함수 활용
  - 소수점, 음수 등 허용 범위를 명시적으로 지정

---

## 5. 의존성 보안

### 필수 준수 사항

- [ ] **`npm audit` 정기 실행**
  - PR 머지 전, 배포 전에 `npm audit`을 실행하여 알려진 취약점 확인
  - `critical` 및 `high` 취약점은 즉시 수정
  - 수정 불가능한 경우 대체 패키지 검토 또는 예외 사유 문서화

- [ ] **불필요한 의존성 제거**
  - 사용하지 않는 패키지는 `package.json`에서 제거
  - 번들 크기와 공격 표면을 최소화
  - `depcheck` 등의 도구로 미사용 의존성 확인

- [ ] **`package-lock.json` 커밋 필수**
  - 의존성 버전을 정확히 고정하여 재현 가능한 빌드 보장
  - `package-lock.json`이 `.gitignore`에 포함되어 있지 않은지 확인
  - CI/CD에서 `npm ci`를 사용하여 lock 파일 기준으로 설치

---

## 6. 스토리지 보안

### 저장 위치 규칙

| 데이터 | 허용 스토리지 | 금지 스토리지 | 이유 |
|---|---|---|---|
| Access Token | 메모리 (JS 변수) | localStorage, sessionStorage, cookie | XSS 시 탈취 방지 |
| Refresh Token | Zustand persist (rememberMe에 따라 결정) | 하드코딩, 전역 변수 | 사용자 선택 존중 |
| RSA 공개키 | 메모리 + sessionStorage | localStorage | 탭 닫으면 삭제되어야 함 |
| 개발 토큰 (`dev-token-*`) | 개발 환경만 | 프로덕션 환경 | 자동 비활성화 확인 |

### 필수 준수 사항

- [ ] **Access Token은 메모리에만 저장**
  - `setAccessToken()` 함수만 사용
  - `localStorage.setItem('accessToken', ...)` 같은 직접 저장 절대 금지

- [ ] **Refresh Token은 Zustand persist로만 관리**
  - `rememberMe=true`: localStorage로 persist
  - `rememberMe=false`: sessionStorage로 persist
  - 직접 스토리지 조작 금지, 반드시 Zustand store를 통해 접근

- [ ] **RSA 공개키는 sessionStorage에 캐시**
  - 키: `rsa-public-key`
  - 탭을 닫으면 자동 삭제된다
  - localStorage에 저장하지 않는다

- [ ] **개발 토큰 프로덕션 비활성화 확인**
  - `dev-token-*` 패턴의 개발용 토큰이 존재하는 경우
  - `isDevToken()` 함수가 프로덕션 환경에서 이를 자동 비활성화하는지 확인
  - 프로덕션 빌드에 개발 토큰 로직이 포함되지 않도록 주의

---

## 7. WebSocket 보안 (webadmin)

> 이 섹션은 webadmin 프로젝트에만 해당한다. appuser에서는 해당 없음.

### 필수 준수 사항

- [ ] **인증 토큰 전달**
  - WebSocket 연결 시 인증 토큰을 포함하여 서버에서 사용자를 식별
  - 토큰 만료 시 재연결 처리

- [ ] **재연결 시 Exponential Backoff**
  - 연결이 끊어졌을 때 즉시 재연결하지 않는다
  - 재연결 간격을 점진적으로 증가시킨다 (1초 → 2초 → 4초 → 8초 → 최대 30초)
  - 서버 부하를 방지하고 네트워크 안정성을 확보

- [ ] **메시지 유효성 검증**
  - 서버에서 수신한 WebSocket 메시지의 형식과 내용을 검증
  - 예상치 못한 메시지 형식에 대한 방어적 처리
  - JSON 파싱 실패 시 에러 핸들링

- [ ] **컴포넌트 unmount 시 연결 정리**
  - WebSocket을 사용하는 컴포넌트가 unmount될 때 반드시 연결을 닫는다
  - `useEffect`의 cleanup 함수에서 `socket.close()` 호출
  - 메모리 누수와 불필요한 연결 방지

```
[체크포인트] WebSocket 코드 리뷰 시 확인할 사항:
  1. 연결 시 인증 토큰이 전달되는가?
  2. 재연결 로직에 backoff가 적용되어 있는가?
  3. 수신 메시지를 파싱/검증하고 있는가?
  4. cleanup 함수에서 연결을 닫고 있는가?
```

---

## 요약: 보안 체크 우선순위

| 우선순위 | 항목 | 영향도 |
|---|---|---|
| P0 (필수) | Access Token 메모리 저장 | 토큰 탈취 시 계정 탈취 |
| P0 (필수) | XSS 방지 (dangerouslySetInnerHTML 금지) | 임의 스크립트 실행 |
| P0 (필수) | 민감 정보 Git 커밋 금지 | 키/토큰 유출 |
| P0 (필수) | HTTPS 강제 | 통신 데이터 도청 |
| P1 (중요) | 입력 검증 (Zod) | 잘못된 데이터 전송 |
| P1 (중요) | 의존성 취약점 검사 | 알려진 취약점 악용 |
| P1 (중요) | 콘솔 로그 민감 정보 제거 | 디버깅 시 정보 노출 |
| P2 (권장) | 소스맵 비포함 | 소스코드 역분석 |
| P2 (권장) | 불필요한 의존성 제거 | 공격 표면 축소 |
