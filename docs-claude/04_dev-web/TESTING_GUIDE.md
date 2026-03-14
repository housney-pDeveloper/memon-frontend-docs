# 테스트 가이드

> **프로젝트**: 31_webadmin (관리자 대시보드)
> **스택**: Next.js 15, Full FSD, Radix UI, shadcn/ui, DataGrid

---

## 1. 테스트 도구

| 용도 | 도구 |
|------|------|
| 단위 테스트 | Vitest + @testing-library/react |
| E2E 테스트 | Playwright |
| DOM 유틸 | @testing-library/jest-dom |

---

## 2. 테스트 파일 위치

| 종류 | 경로 패턴 |
|------|-----------|
| 단위 테스트 | `src/{layer}/{domain}/{segment}/__tests__/*.test.ts` |
| E2E 테스트 | `e2e/*.spec.ts` |

---

## 3. 단위 테스트 작성 패턴

```typescript
import { describe, it, expect, vi } from 'vitest'
import { renderHook, act } from '@testing-library/react'
import { render, screen } from '@testing-library/react'

// 훅 테스트
describe('useLogin', () => {
  it('로그인 성공 시 토큰을 저장한다', async () => {
    // Arrange - Act - Assert 패턴
  })
})

// 컴포넌트 테스트
describe('UserCard', () => {
  it('사용자 이름을 표시한다', () => {
    render(<UserCard user={mockUser} />)
    expect(screen.getByText('홍길동')).toBeInTheDocument()
  })
})
```

---

## 4. 테스트 우선순위

| 우선순위 | 대상 | 목표 커버리지 |
|----------|------|---------------|
| 1 | 유틸리티 함수 (`shared/lib/`) | 90%+ |
| 2 | 커스텀 훅 (`model/use*.ts`) | 80%+ |
| 3 | API 함수 | 70%+ |
| 4 | 컴포넌트 | 60%+ |

---

## 5. 실행 명령어

```bash
npm run test        # watch 모드
npm run test:run    # 1회 실행
npm run e2e         # Playwright E2E
```

---

## 6. 목업 전략

| 대상 | 방법 |
|------|------|
| API | `vi.spyOn` / `vi.mock` |
| TanStack Query | `QueryClientProvider` 래퍼 |
| Zustand | 초기 상태 주입 |
| Router | Next.js navigation 모킹 |
