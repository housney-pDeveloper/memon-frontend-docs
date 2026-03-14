# 32_appuser 코드 컨벤션

> Next.js 15 | Light FSD | 순수 Tailwind CSS | Capacitor v1 → React Native v2 전환 예정

---

## 1. 파일 명명 규칙

webadmin과 동일한 규칙을 따른다.

| 구분 | 네이밍 | 예시 |
|------|--------|------|
| 컴포넌트 | `PascalCase.tsx` | `LoginForm.tsx` |
| API | `camelCase.ts` | `authApi.ts` |
| 유틸리티 | `camelCase.ts` | `formatDate.ts` |
| 스토어 | `camelCase.ts` | `auth.store.ts` |
| 훅 | `camelCase.ts`, `use` 접두사 | `useLogin.ts` |
| 타입 | `camelCase.ts` | `types.ts` |

---

## 2. 컴포넌트 작성 규칙

webadmin과 동일한 규칙을 따른다.

```tsx
interface LoginFormProps {
  onSuccess?: () => void
}

export const LoginForm = ({ onSuccess }: LoginFormProps) => {
  return <div>...</div>
}
LoginForm.displayName = "LoginForm"
```

- Arrow Function 사용
- Props 인터페이스 분리 선언
- displayName 필수

---

## 3. 핵심 차이점 — Radix UI 사용 금지

```tsx
// X: Radix UI 사용 금지
import { Dialog } from '@radix-ui/react-dialog'

// O: 순수 Tailwind CSS로 구현
export const Popup = ({ isOpen, onClose, children }: PopupProps) => {
  if (!isOpen) return null
  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      <div className="fixed inset-0 bg-black/50" onClick={onClose} />
      <div className="relative bg-white rounded-lg p-6">{children}</div>
    </div>
  )
}
```

- **이유**: React Native v2 전환 시 Radix UI 호환 불가
- `shared/ui/`: 순수 Tailwind CSS만 사용하여 컴포넌트 구현
- `shared/primitives/` 디렉토리 없음

---

## 4. core/ 레이어 사용 필수

플랫폼 의존적인 API는 반드시 `core/` 레이어를 통해 접근한다.

```typescript
// O: core/ 레이어를 통한 접근
import { authStorage } from '@/core/storage'
await authStorage.setToken(token)
await authStorage.getToken()

// X: 직접 접근 금지 (v2에서 전부 수정 필요)
localStorage.setItem('token', token)
localStorage.getItem('token')
window.navigator.geolocation
```

---

## 5. import 순서

webadmin과 동일한 규칙을 따른다.

```typescript
// 1. React/Next.js
import { useState } from 'react'
import { useRouter } from 'next/navigation'

// 2. 외부 라이브러리
import { useQuery } from '@tanstack/react-query'

// 3. hs-common (@hs/common/*)
import { ApiError } from '@hs/common/api'

// 4. 내부 모듈 (@/core/, @/features/, @/shared/)
import { authStorage } from '@/core/storage'
import { useAuth } from '@/features/auth/hooks/useAuth'
import { Button } from '@/shared/ui/Button'

// 5. 타입 (type-only import)
import type { LoginResponse } from '@/features/auth/types'
```

---

## 6. API 함수 작성

- `buildApiUrl()` 사용 → `/app/v1/...`
- `@hs/common/api`에서 에러 유틸 import
- `@hs/common/types`에서 응답 타입 import

```typescript
import { buildApiUrl } from '@/shared/lib/api'
import { handleApiError } from '@hs/common/api'
import type { ApiResponse } from '@hs/common/types'

export const fetchUserProfile = async (): Promise<UserProfile> => {
  const url = buildApiUrl('/app/v1/users/me')
  // ...
}
```

---

## 7. 금지 사항

| 금지 항목 | 이유 |
|-----------|------|
| `any` 타입 사용 | 타입 안전성 저하 |
| `console.log` 커밋 | 프로덕션 로그 오염 |
| Radix UI / shadcn 사용 | React Native v2 전환 호환 불가 |
| 직접 `localStorage` / `sessionStorage` / `window` 접근 | `core/` 레이어를 통해야 v2 전환 가능 |
| barrel import (`index.ts`에서 re-export) | 번들 크기 및 순환 참조 문제 |
| `function` 키워드 컴포넌트 선언 | Arrow Function 통일 |
