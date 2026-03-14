# 테스트 표준

> 프론트엔드 프로젝트(webadmin, appuser, hs-common) 전체에 적용되는 테스트 표준.

---

## 1. 테스트 환경

| 프로젝트 | 단위 테스트 | E2E 테스트 | 현재 상태 |
|----------|------------|------------|-----------|
| 31_webadmin | Vitest + Testing Library | Playwright | 활성 |
| 32_appuser | - | - | 미구성 |
| 21_hs-common | - | - | 호스트 프로젝트에서 테스트 |

## 2. 단위 테스트 표준

### 테스트 대상 우선순위

1. 유틸리티 함수 (`shared/lib/`) — 목표 커버리지: 90%+
2. 커스텀 훅 (`model/use*.ts`) — 목표: 80%+
3. API 함수 (`api/*.ts`) — 목표: 70%+
4. 컴포넌트 (`ui/*.tsx`) — 목표: 60%+

### 테스트 파일 위치

- 테스트 대상과 같은 디렉토리의 `__tests__/` 하위
- 테스트 파일명: `{대상파일명}.test.ts` 또는 `.test.tsx`

### 작성 패턴: Arrange-Act-Assert (AAA)

```typescript
describe('함수/훅/컴포넌트명', () => {
  it('기대 동작을 한글로 서술한다', () => {
    // Arrange: 테스트 데이터 준비

    // Act: 실행

    // Assert: 검증
  })
})
```

## 3. E2E 테스트 표준

### 테스트 대상: 핵심 사용자 시나리오

- 로그인/로그아웃
- 주요 CRUD 플로우
- 권한 기반 접근 제어
- 에러 시나리오

### 파일 구조

- 테스트 파일 위치: `e2e/` 디렉토리
- 시나리오별 디렉토리 분리: `e2e/auth/`, `e2e/member/` 등

## 4. 실행 명령어

```bash
# webadmin
npm run test          # Vitest watch 모드
npm run test:run      # Vitest 1회 실행
npm run e2e           # Playwright E2E

# 타입 체크 (테스트는 아니지만 품질 게이트)
npm run type-check
npm run lint
```

## 5. 목업 전략

- **API**: `vi.spyOn` 또는 `vi.mock` (실제 네트워크 호출 금지)
- **TanStack Query**: 테스트용 QueryClient + QueryClientProvider 래퍼
- **Zustand**: `store.setState()` 또는 초기 상태 주입
- **Next.js Router**: `next/navigation` 모킹

## 6. 품질 게이트 (CI/CD 통과 조건)

- [ ] `npm run type-check` 통과
- [ ] `npm run lint` 통과
- [ ] `npm run test:run` 전체 통과
- [ ] `npm run e2e` 전체 통과 (webadmin)
