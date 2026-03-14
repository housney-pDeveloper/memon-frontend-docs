# API 컨벤션

## 1. Gateway 패턴

모든 API URL은 다음 형식을 따른다:

```
/{clientType}/{version}/{endpoint}
```

- **webadmin**: `clientType = 'web'` → `/web/v1/login`
- **appuser**: `clientType = 'app'` → `/app/v1/login`

### 구현 방식

`@hs/common/api`의 `createApiConfig()`를 프로젝트별 `buildApiUrl()`로 래핑한다.

- **webadmin**: `shared/config/api.ts`

  ```typescript
  createApiConfig({ clientType: 'web', version: 'v1' })
  ```

- **appuser**: `shared/config/api-config.ts`

  ```typescript
  createApiConfig({ clientType: 'app', version: 'v1' })
  ```

## 2. Axios 클라이언트

파일 위치: `shared/api/client.ts`

```typescript
apiClient = axios.create({ baseURL: ENV.API_BASE_URL, timeout: 30000 })
```

### 요청 인터셉터

- `Authorization: Bearer {token}` 자동 주입

### 응답 인터셉터

| 조건 | 처리 |
|------|------|
| `success === false` | Gateway 에러 throw |
| `code !== 1 (SUCCESS)` | Application 에러 throw |
| 401 / JWT 에러 | 토큰 리프레시 시도 |
| 403 | `/error/403` 리다이렉트 |
| 네트워크 에러 | `NETWORK_ERROR` 코드 반환 |

### FSD 콜백 패턴

`setApiClientAuthCallbacks()`를 사용하여 app 레이어에서 인증 콜백을 주입한다.

## 3. FSD 기반 API 배치 규칙

| 분류 | HTTP Method | FSD 위치 | 예시 |
|------|-------------|----------|------|
| 조회 (read-only) | GET | `entities/{domain}/api/{domain}Api.ts` | `getItemList()`, `getItemDetail()` |
| 생성/수정/삭제 | POST/PUT/PATCH/DELETE | `features/{domain}/api/{domain}Api.ts` | `createItem()`, `updateItem()`, `deleteItem()` |

## 4. API 함수 작성 패턴

### entities (GET only)

```typescript
import { apiClient } from '@/shared/api/client'
import { buildApiUrl } from '@/shared/config/api'
import type { ServerResponse } from '@hs/common/types'

export const getItems = async (params: ListParams): Promise<ServerResponse<Item[]>> => {
  const { data } = await apiClient.get(buildApiUrl('/items'), { params })
  return data
}
```

### features (POST/PUT/PATCH/DELETE)

```typescript
export const createItem = async (payload: CreatePayload): Promise<ServerResponse<void>> => {
  const { data } = await apiClient.post(buildApiUrl('/items'), payload)
  return data
}
```

## 5. TanStack Query 훅 패턴

### Query 훅 (entities)

파일 위치: `entities/{domain}/model/useGetItems.ts`

```typescript
export const useGetItems = (params: ListParams) => {
  return useQuery({
    queryKey: ['items', 'list', params],
    queryFn: () => getItems(params),
  })
}
```

### Mutation 훅 (features)

파일 위치: `features/{domain}/model/useCreateItem.ts`

```typescript
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

## 6. 환경 변수

- Next.js에서는 `NEXT_PUBLIC_` prefix를 사용한다.
- `shared/config/env.ts`에서 `ENV` 객체로 타입 안전하게 래핑한다.
- 기본값: `API_BASE_URL = 'http://localhost:9999'`

### 환경 파일

| 파일 | 용도 |
|------|------|
| `.env` | 로컬 |
| `.env.dev` | 개발 |
| `.env.prod` | 운영 |
