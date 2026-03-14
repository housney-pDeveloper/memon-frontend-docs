# 코드 컨벤션

> **프로젝트**: 31_webadmin (관리자 대시보드)
> **스택**: Next.js 15, Full FSD, Radix UI, shadcn/ui, DataGrid

---

## 1. 파일 명명 규칙

| 분류 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | `PascalCase.tsx` | `UserCard.tsx`, `LoginForm.tsx` |
| API 파일 | `camelCase.ts` | `authApi.ts`, `memberApi.ts` |
| 유틸리티 | `camelCase.ts` | `utils.ts`, `format.ts`, `validation.ts` |
| 스토어 | `camelCase.ts` | `store.ts`, `signupStore.ts` |
| 훅 | `camelCase.ts`, `use` 접두사 | `useLogin.ts`, `useGetMe.ts` |
| 타입 | `camelCase.ts` | `types.ts` |
| 스타일 | `camelCase.variants.ts` | `button.variants.ts` |
| 상수 | `camelCase.ts` | `permissions.ts`, `pagination.ts` |

---

## 2. 컴포넌트 작성 규칙

```tsx
// 1. Props 인터페이스 분리 선언
interface UserCardProps {
  user: User
  onEdit?: (id: string) => void
}

// 2. Arrow Function 사용 (function 선언 금지)
export const UserCard = ({ user, onEdit }: UserCardProps) => {
  return <Card>...</Card>
}

// 3. displayName 필수
UserCard.displayName = "UserCard"
```

- Props 인터페이스: `{ComponentName}Props` 형식으로 명명
- 이벤트 핸들러 Props: `on` + 동사 (예: `onEdit`, `onDelete`, `onSubmit`)
- 내부 핸들러 함수: `handle` + 동사 (예: `handleClick`, `handleSubmit`)

---

## 3. import 순서

```typescript
// 1. React/Next.js
import { useState, useEffect } from 'react'
import { useRouter } from 'next/navigation'

// 2. 외부 라이브러리
import { useQuery } from '@tanstack/react-query'
import { useForm } from 'react-hook-form'

// 3. hs-common
import { getApiErrorMessage } from '@hs/common/api'
import type { ServerResponse } from '@hs/common/types'

// 4. 내부 모듈 (FSD 레이어 순서)
import { useGetMe } from '@/entities/auth/model/useGetMe'
import { apiClient } from '@/shared/api/client'
import { Button } from '@/shared/ui'

// 5. 타입 (type-only import)
import type { User } from '@/entities/auth/model/types'
```

---

## 4. 변수/상수 명명

| 분류 | 규칙 | 예시 |
|------|------|------|
| 변수/함수 | `camelCase` | `userName`, `getUserList` |
| 상수 | `UPPER_SNAKE_CASE` | `API_BASE_URL`, `AUTH_STORAGE_KEY` |
| 타입/인터페이스 | `PascalCase` | `User`, `LoginFormProps` |
| Enum 값 | `UPPER_SNAKE_CASE` | `USER_ROLE`, `AUTH_STATUS` |
| Boolean 변수 | `is`/`has`/`can` 접두사 | `isLoading`, `hasPermission`, `canEdit` |

---

## 5. API 함수 작성

- **URL 구성**: `buildApiUrl()` 필수 사용 (직접 URL 문자열 조합 금지)
- **응답 타입**: `ServerResponse<T>` 사용
- **에러 처리**: `getApiErrorMessage()` / `parseApiError()` from `@hs/common/api`

---

## 6. 금지 사항

| 금지 항목 | 이유 |
|-----------|------|
| `any` 타입 사용 | 타입 안전성 파괴, `unknown` 또는 구체 타입 사용 |
| `console.log` 커밋 | 디버깅 후 반드시 제거 |
| 인라인 스타일 사용 | Tailwind CSS 사용 |
| barrel import | `@/features/auth` 금지 → `@/features/auth/model/useLogin` 사용 |
| `function` 키워드 컴포넌트 선언 | Arrow Function 사용 |
