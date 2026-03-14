# 타입 컨벤션

## 1. 공유 타입

패키지 위치: `@hs/common/types`

```typescript
interface ServerResponse<T = unknown> {
  code: number
  message: string
  data?: T | null
  success?: boolean  // Gateway 에러 시 false
}

interface ApiError {
  code: number | string
  message: string
  details?: unknown
  status?: number
}

interface PaginatedResponse<T> {
  data: T[]
  total: number
  page: number
  pageSize: number
  totalPages: number
}
```

## 2. 타입 명명 규칙

| 분류 | 규칙 | 예시 |
|------|------|------|
| 타입/인터페이스 | `PascalCase` | `User`, `LoginRequest`, `ItemListResponse` |
| Props | `{ComponentName}Props` | `UserCardProps`, `LoginFormProps` |
| API 요청 | `{Action}{Domain}Request` | `CreateItemRequest` |
| API 응답 | `{Domain}{Type}Response` | `ItemListResponse`, `ItemDetailResponse` |
| 스토어 상태 | `{Domain}State` 또는 `{Domain}Store` | `AuthState` |
| Zod 스키마 | `{name}Schema` | `loginSchema` |
| Zod 타입 추론 | `z.infer<typeof schema>` | `z.infer<typeof loginSchema>` |

## 3. 타입 배치 위치

| 타입 분류 | 위치 |
|-----------|------|
| 도메인 엔티티 타입 | `entities/{domain}/model/types.ts` |
| 공유 타입 | `shared/types/*.ts` |
| Props | 컴포넌트 파일 상단에 인터페이스 선언 |
| API 요청/응답 타입 | API 파일 또는 별도 `types.ts` |

## 4. 금지 사항

- **`any` 타입 사용 금지**: 반드시 구체적인 타입 또는 `unknown`을 사용한다.
- **`as` 타입 단언 최소화**: 타입 단언보다 type narrowing을 우선 사용한다.
- **불필요한 타입 래핑 금지**: `type Foo = string` 같은 trivial alias를 만들지 않는다.
