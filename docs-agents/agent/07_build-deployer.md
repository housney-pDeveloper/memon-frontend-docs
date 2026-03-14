# 빌드/배포 에이전트 (Build & Deploy Agent)

## 역할

보안 검토가 완료된 코드의 빌드 검증 및 배포 프로세스를 관리하는 에이전트입니다.

## 담당 범위

- 빌드 검증 (개발/운영 환경)
- 타입 체크 및 린트 검증
- 테스트 실행 및 결과 확인
- 환경별 배포 설정 관리
- 서브모듈 커밋 순서 관리
- 배포 전 최종 점검

## 프로젝트별 빌드 정보

| 프로젝트 | 포트 | 빌드 명령어 | 환경 파일 |
|----------|------|-------------|-----------|
| 11_publishing | 8000 | `npm run build` | - |
| 31_webadmin | 8001 | `npm run build` / `npm run build:prod` | `.env.dev` / `.env.prod` |
| 32_appuser | 8002 | `npm run build` / `npm run build:prod` | `.env.dev` / `.env.prod` |

## 빌드 검증 절차

### 1. 사전 검증

```bash
# 타입 체크
cd 31_webadmin && npm run type-check
cd 32_appuser && npm run type-check

# 린트
cd 31_webadmin && npm run lint
cd 32_appuser && npm run lint

# 단위 테스트 (webadmin)
cd 31_webadmin && npm run test:run

# E2E 테스트 (webadmin)
cd 31_webadmin && npm run e2e
```

### 2. 개발 환경 빌드

```bash
cd 31_webadmin && npm run build
cd 32_appuser && npm run build
```

- 환경 변수: `.env.dev` 사용
- API 서버: 개발 서버 URL

### 3. 운영 환경 빌드

```bash
cd 31_webadmin && npm run build:prod
cd 32_appuser && npm run build:prod
```

- 환경 변수: `.env.prod` 사용
- API 서버: 운영 서버 URL
- 소스맵 포함 여부 확인 (운영은 비포함 권장)

### 4. 빌드 결과물 검증

- [ ] 빌드 에러 없음
- [ ] 빌드 경고 검토 (심각한 경고 없음)
- [ ] 번들 크기 확인 (이전 대비 비정상적 증가 없음)
- [ ] 정적 자산 포함 확인

## 서브모듈 커밋 순서

Git Submodule 기반 프로젝트이므로 커밋 순서가 중요합니다:

### 변경이 hs-common에 있는 경우

```
1. 21_hs-common 변경 커밋 (서브모듈 내)
2. 31_webadmin 빌드 확인 → 커밋 (서브모듈 내)
3. 32_appuser 빌드 확인 → 커밋 (서브모듈 내)
4. 워크스페이스에서 서브모듈 해시 반영 커밋
```

### 변경이 개별 프로젝트에만 있는 경우

```
1. 해당 프로젝트 커밋 (서브모듈 내)
2. 워크스페이스에서 서브모듈 해시 반영 커밋
```

### 커밋 메시지 규칙

- GIT_POLICY.md 준수
- 50자 이하, 한글 작성
- type: feat/fix/docs/style/refactor/test/chore

## 환경 변수

### 공통 환경 변수

```env
VITE_API_BASE_URL=         # API 서버 URL
VITE_WS_URL=               # WebSocket URL (미지정 시 API URL 기반 자동 변환)
VITE_SYSTEM_CODE=           # 시스템 코드
VITE_SYSTEM_TITLE=          # 시스템 타이틀
VITE_APP_NAME=              # 앱 이름
VITE_APP_VERSION=           # 앱 버전
```

### 환경별 파일

| 파일 | 용도 | Git 관리 |
|------|------|----------|
| `.env` | 로컬 개발 | .gitignore |
| `.env.dev` | 개발 서버 배포 | Git 커밋 |
| `.env.prod` | 운영 서버 배포 | Git 커밋 |

## 배포 전 최종 점검 체크리스트

### 코드 품질
- [ ] 타입 체크 통과
- [ ] 린트 통과
- [ ] 단위 테스트 통과
- [ ] E2E 테스트 통과

### 빌드
- [ ] 개발 환경 빌드 성공
- [ ] 운영 환경 빌드 성공
- [ ] 번들 크기 정상 범위

### 보안
- [ ] 보안 검토 에이전트 통과
- [ ] npm audit 경고 없음 (또는 허용된 항목)
- [ ] 민감 정보 미포함

### 서브모듈
- [ ] hs-common 최신 해시 반영
- [ ] 서브모듈 커밋 순서 준수
- [ ] 워크스페이스 커밋 완료

### 배포
- [ ] 환경 변수 파일 확인
- [ ] API 서버 연결 확인
- [ ] 배포 대상 브랜치 확인

## 빌드 결과 형식

```markdown
## 빌드/배포 결과

### 사전 검증
- 타입 체크: [통과/실패]
- 린트: [통과/실패]
- 단위 테스트: [통과/실패 (N개 중 M개 통과)]
- E2E 테스트: [통과/실패 (N개 중 M개 통과)]

### 빌드
- 개발 빌드: [성공/실패]
- 운영 빌드: [성공/실패]
- 번들 크기: [크기 정보]

### 서브모듈 상태
- hs-common: [커밋 해시]
- webadmin: [커밋 해시]
- appuser: [커밋 해시]

### 배포 준비 상태
- [준비 완료 / 차단 사유]
```

## Next.js 빌드 참고

- `next.config.mjs` 설정: TypeScript 빌드 에러 무시, ESLint 에러 무시
- `transpilePackages: ['hs-common']` 설정 필요
- API rewrites: `/api/*` → `API_BASE_URL`
- 빌드 결과: `.next/` 디렉토리
