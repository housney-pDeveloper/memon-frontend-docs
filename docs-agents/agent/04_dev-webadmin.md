# 웹 관리자 개발 에이전트 (Webadmin Developer Agent)

## 역할

설계 문서를 기반으로 `31_webadmin` 프로젝트의 기능을 구현하는 에이전트입니다.

## 담당 범위

- `31_webadmin/src/` 내 코드 구현
- FSD(Feature-Sliced Design) 아키텍처 준수
- webadmin 전용 UI 컴포넌트 구현 (Radix UI, shadcn/ui, DataGrid)

## 프로젝트 정보

- **경로**: `31_webadmin/`
- **포트**: 8001
- **API clientType**: `web` → URL: `/web/v1/...`
- **아키텍처**: Full FSD
- **UI 라이브러리**: Radix UI, shadcn/ui, Lucide React
- **데이터 그리드**: TanStack Table (DataGrid 컴포넌트)
- **테스트**: Vitest(단위), Playwright(E2E)

## 코딩 컨벤션

### 파일 명명

- 컴포넌트: `PascalCase.tsx` (예: `UserCard.tsx`)
- API: `camelCase.ts` (예: `authApi.ts`)
- 유틸리티: `camelCase.ts`
- 타입: `PascalCase` (예: `User`, `LoginRequest`)
- 상수: `UPPER_SNAKE_CASE` (예: `API_BASE_URL`)

### 컴포넌트 작성 규칙

```tsx
interface ComponentNameProps {
  prop1: string
  onAction?: (id: string) => void
}

export const ComponentName = ({ prop1, onAction }: ComponentNameProps) => {
  return <div>...</div>
}
ComponentName.displayName = "ComponentName"
```

- Arrow Function 사용
- Props 인터페이스 분리 선언
- displayName 필수
- 이벤트 핸들러: `on` + 동사 (예: `onEdit`, `onDelete`)

### API 작성 규칙

```typescript
// entities (GET만)
import { apiClient } from '@/shared/api/client'
import { buildApiUrl } from '@/shared/config/api'
import type { ServerResponse } from '@hs/common/types'

export const getItems = async (params: ListParams): Promise<ServerResponse<Item[]>> => {
  const { data } = await apiClient.get(buildApiUrl('items'), { params })
  return data
}

// features (POST/PUT/PATCH/DELETE)
export const createItem = async (payload: CreatePayload): Promise<ServerResponse<void>> => {
  const { data } = await apiClient.post(buildApiUrl('items'), payload)
  return data
}
```

### TanStack Query 훅

```typescript
// entities/{domain}/model/useGetItems.ts
export const useGetItems = (params: ListParams) => {
  return useQuery({
    queryKey: ['items', 'list', params],
    queryFn: () => getItems(params),
  })
}

// features/{domain}/model/useCreateItem.ts
export const useCreateItem = () => {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: createItem,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['items'] })
    },
  })
}
```

### 폼 작성 규칙

```typescript
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  name: z.string().min(1, '이름을 입력하세요'),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
})

type FormValues = z.infer<typeof schema>
```

## FSD 의존성 규칙

```
app → widgets → features → entities → shared
```

- entities 간 교차 참조 금지
- features 간 교차 참조 금지
- shared에서 entities 참조 금지
- barrel import 금지 (features/index.ts, entities/index.ts)
- primitives는 shared/ui 내에서만 import

## 구현 절차

1. **타입 정의**: `entities/{domain}/model/types.ts`
2. **API 함수**: entities(GET) → features(변경)
3. **쿼리 훅**: `useGet...`, `useCreate...` 등
4. **UI 컴포넌트**: entities/ui(표시) → features/ui(기능)
5. **페이지**: `app/` 라우트에 페이지 추가
6. **가드/권한**: `PermissionGuard`, `ProtectedRoute` 적용
7. **메뉴 등록**: `shared/config/menu-config.ts` 업데이트

## 사용 가능한 공통 컴포넌트

| 컴포넌트 | 위치 | 용도 |
|----------|------|------|
| Button | `shared/ui/button.tsx` | 버튼 (variants 지원) |
| Input | `shared/ui/input.tsx` | 입력 필드 |
| Card | `shared/ui/card.tsx` | 카드 레이아웃 |
| Popup | `shared/ui/popup.tsx` | 모달/다이얼로그 |
| DataGrid | `shared/components/data-grid.tsx` | 데이터 테이블 |
| AutoComplete | `shared/components/auto-complete.tsx` | 검색+선택 |
| TreeSearch | `shared/components/TreeSearch.tsx` | 트리 구조 검색 |
| ErrorBoundary | `shared/components/ErrorBoundary.tsx` | 에러 경계 |

## 에러 처리

```typescript
import { getApiErrorMessage, parseApiError } from '@hs/common/api'

// 뮤테이션 에러
onError: (error) => {
  toast.error(getApiErrorMessage(error, '처리 중 오류가 발생했습니다'))
}
```

- 에러 메시지는 서버 응답 그대로 사용
- Gateway JWT 오류(11XXX), 인증 오류(92XXX)는 자동 리다이렉트

## 스크립트

```bash
npm run dev          # 개발 서버 (localhost:8001)
npm run build        # 개발 빌드
npm run build:prod   # 운영 빌드
npm run test         # Vitest 단위 테스트 (watch)
npm run test:run     # Vitest 단위 테스트 (1회)
npm run e2e          # Playwright E2E 테스트
npm run type-check   # TypeScript 타입 체크
npm run lint         # ESLint 실행
```
