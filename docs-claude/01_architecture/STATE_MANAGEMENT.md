# 상태 관리 가이드

## 개요

본 프로젝트는 상태의 성격에 따라 4가지 상태 관리 전략을 사용한다.

| 상태 유형 | 도구 | 용도 |
|----------|------|------|
| 서버 상태 | TanStack Query (React Query) | API 데이터 페칭/캐싱 |
| 클라이언트 상태 | Zustand | UI 상태, 인증 상태 |
| 폼 상태 | React Hook Form + Zod | 폼 입력/유효성 검증 |
| URL 상태 | Next.js searchParams | 필터, 페이지네이션, 탭 |

---

## 1. 서버 상태 — TanStack Query (React Query)

### 원칙

- **모든 API 데이터 페칭과 캐싱은 TanStack Query를 사용한다.**
- 직접 `useEffect` + `useState`로 API를 호출하지 않는다.
- 서버 데이터는 클라이언트 스토어(Zustand)에 복제하지 않는다.

### 쿼리 키 컨벤션

쿼리 키는 배열 형태로 구성하며, `['도메인', '타입', 파라미터]` 패턴을 따른다.

```typescript
// 목록 조회
['items', 'list', { page: 1, size: 20, status: 'active' }]

// 상세 조회
['items', 'detail', '123']

// 전체 목록 (파라미터 없음)
['items', 'list']

// 하위 리소스
['items', 'comments', { itemId: '123' }]
```

**쿼리 키 네이밍 규칙:**

| 타입 | 키 패턴 | 예시 |
|------|---------|------|
| 목록 | `['domain', 'list', params]` | `['items', 'list', { page: 1 }]` |
| 상세 | `['domain', 'detail', id]` | `['items', 'detail', '123']` |
| 무한 스크롤 | `['domain', 'infinite', params]` | `['items', 'infinite', { category: 'A' }]` |
| 집계/통계 | `['domain', 'stats', params]` | `['items', 'stats', { period: 'monthly' }]` |

### 파일 위치 (webadmin FSD 기준)

#### 쿼리 훅 — `entities/{domain}/model/`

읽기 전용 데이터 조회 훅은 entities 레이어에 위치한다.

```typescript
// entities/item/model/useGetItemList.ts
import { useQuery } from '@tanstack/react-query';
import { getItemList } from '@/entities/item/api/getItemList';

interface UseGetItemListParams {
  page: number;
  size: number;
  status?: string;
}

export function useGetItemList(params: UseGetItemListParams) {
  return useQuery({
    queryKey: ['items', 'list', params],
    queryFn: () => getItemList(params),
  });
}
```

```typescript
// entities/item/model/useGetItemDetail.ts
import { useQuery } from '@tanstack/react-query';
import { getItemDetail } from '@/entities/item/api/getItemDetail';

export function useGetItemDetail(id: string) {
  return useQuery({
    queryKey: ['items', 'detail', id],
    queryFn: () => getItemDetail(id),
    enabled: !!id,
  });
}
```

#### 뮤테이션 훅 — `features/{domain}/model/`

데이터 변경 훅은 features 레이어에 위치한다.

```typescript
// features/item/model/useCreateItem.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { createItem } from '@/features/item/api/createItem';

export function useCreateItem() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: createItem,
    onSuccess: () => {
      // 관련 쿼리 무효화 — 목록 자동 갱신
      queryClient.invalidateQueries({ queryKey: ['items', 'list'] });
    },
  });
}
```

```typescript
// features/item/model/useUpdateItem.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { updateItem } from '@/features/item/api/updateItem';

export function useUpdateItem() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: updateItem,
    onSuccess: (_, variables) => {
      // 목록과 해당 상세 쿼리 모두 무효화
      queryClient.invalidateQueries({ queryKey: ['items', 'list'] });
      queryClient.invalidateQueries({ queryKey: ['items', 'detail', variables.id] });
    },
  });
}
```

```typescript
// features/item/model/useDeleteItem.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { deleteItem } from '@/features/item/api/deleteItem';

export function useDeleteItem() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: deleteItem,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['items', 'list'] });
    },
  });
}
```

### 쿼리 무효화 규칙

뮤테이션 성공 시 **반드시** 관련 쿼리를 무효화하여 UI를 최신 상태로 유지한다.

| 뮤테이션 유형 | 무효화 대상 |
|-------------|-----------|
| 생성 (Create) | 목록 쿼리 |
| 수정 (Update) | 목록 쿼리 + 해당 상세 쿼리 |
| 삭제 (Delete) | 목록 쿼리 |
| 상태 변경 | 목록 쿼리 + 해당 상세 쿼리 + 관련 통계 쿼리 |

### QueryProvider 설정

`app/providers/QueryProvider.tsx`에서 글로벌 설정을 관리한다.

```typescript
// app/providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,       // 5분
      gcTime: 1000 * 60 * 30,          // 30분
      retry: 1,
      refetchOnWindowFocus: false,
    },
  },
});

export function QueryProvider({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

---

## 2. 클라이언트 상태 — Zustand

### 원칙

- **서버에서 오는 데이터는 TanStack Query로, UI/클라이언트 고유 상태만 Zustand로 관리한다.**
- API 응답 데이터를 Zustand에 복제하지 않는다.

### 사용 대상

| 상태 유형 | 예시 |
|----------|------|
| 인증 상태 | 액세스 토큰, 사용자 인증 여부 |
| UI 상태 | 사이드바 토글, 모달 열림/닫힘, 테마 |
| 임시 상태 | 다단계 폼의 스텝 간 데이터 공유 |

### 스토어 위치

| 위치 | 용도 |
|------|------|
| `entities/{domain}/model/store.ts` | 특정 도메인의 클라이언트 상태 |
| `shared/stores/` | 도메인 무관 글로벌 상태 |

### 인증 스토어 패턴

보안을 위해 액세스 토큰과 리프레시 토큰의 저장 방식을 분리한다.

```typescript
// entities/auth/model/authStorage.ts (webadmin 기준)

// 액세스 토큰: 메모리에만 저장 (XSS 방어)
let accessToken: string | null = null;

export const authStorage = {
  getAccessToken: () => accessToken,
  setAccessToken: (token: string) => { accessToken = token; },
  clearAccessToken: () => { accessToken = null; },
};
```

```typescript
// entities/auth/model/store.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  isAuthenticated: boolean;
  user: User | null;
  setAuth: (user: User) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      isAuthenticated: false,
      user: null,
      setAuth: (user) => set({ isAuthenticated: true, user }),
      clearAuth: () => set({ isAuthenticated: false, user: null }),
    }),
    {
      name: 'auth-storage',  // 쿠키/localStorage 키
    }
  )
);
```

**핵심 보안 원칙:**
- **액세스 토큰**: 메모리(변수)에만 저장한다. `localStorage`, 쿠키에 저장하지 않는다.
- **리프레시 토큰**: Zustand persist를 통해 저장한다 (미들웨어가 자동으로 persist 처리).
- 페이지 새로고침 시 리프레시 토큰으로 액세스 토큰을 재발급받는다.

### 셀렉터를 통한 리렌더링 최적화

Zustand 스토어를 사용할 때, 반드시 셀렉터를 사용하여 필요한 상태만 구독한다.

```typescript
// [권장] 셀렉터 사용 — 해당 필드가 변경될 때만 리렌더링
const isAuthenticated = useAuthStore((s) => s.isAuthenticated);
const user = useAuthStore((s) => s.user);

// [금지] 구조 분해 할당 — 스토어 내 어떤 값이든 변경되면 리렌더링
const { isAuthenticated, user } = useAuthStore();
```

**셀렉터 규칙:**

```typescript
// 단일 필드
const theme = useThemeStore((s) => s.theme);

// 다중 필드가 필요한 경우 — 개별 셀렉터 사용
const sidebarOpen = useSidebarStore((s) => s.isOpen);
const sidebarWidth = useSidebarStore((s) => s.width);

// 또는 shallow 비교 사용
import { shallow } from 'zustand/shallow';

const { isOpen, width } = useSidebarStore(
  (s) => ({ isOpen: s.isOpen, width: s.width }),
  shallow
);
```

---

## 3. 폼 상태 — React Hook Form + Zod

### 원칙

- **모든 폼은 React Hook Form + zodResolver를 사용한다.**
- Zod 스키마가 유효성 검증의 단일 소스이다.
- 폼 타입은 스키마에서 추론한다.
- 에러 메시지는 한국어로 작성한다.

### 기본 패턴

```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// 1. Zod 스키마 정의 (유효성 검증 규칙 + 한국어 에러 메시지)
const itemCreateSchema = z.object({
  name: z
    .string()
    .min(1, '상품명을 입력해주세요.')
    .max(100, '상품명은 100자 이내로 입력해주세요.'),
  price: z
    .number({ invalid_type_error: '가격을 입력해주세요.' })
    .min(0, '가격은 0원 이상이어야 합니다.'),
  category: z
    .string()
    .min(1, '카테고리를 선택해주세요.'),
  description: z
    .string()
    .max(500, '설명은 500자 이내로 입력해주세요.')
    .optional(),
  isActive: z.boolean().default(true),
});

// 2. 스키마에서 타입 추론
type ItemCreateFormValues = z.infer<typeof itemCreateSchema>;

// 3. 폼 훅 사용
export function ItemCreateForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<ItemCreateFormValues>({
    resolver: zodResolver(itemCreateSchema),
    defaultValues: {
      name: '',
      price: 0,
      category: '',
      description: '',
      isActive: true,
    },
  });

  const createItem = useCreateItem();

  const onSubmit = (data: ItemCreateFormValues) => {
    createItem.mutate(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* 폼 필드 렌더링 */}
    </form>
  );
}
```

### 에러 메시지 한국어 작성 규칙

| 유효성 검증 | 에러 메시지 패턴 |
|-----------|---------------|
| 필수 입력 | `{필드명}을(를) 입력해주세요.` |
| 필수 선택 | `{필드명}을(를) 선택해주세요.` |
| 최소 길이 | `{필드명}은(는) {N}자 이상 입력해주세요.` |
| 최대 길이 | `{필드명}은(는) {N}자 이내로 입력해주세요.` |
| 최솟값 | `{필드명}은(는) {N} 이상이어야 합니다.` |
| 최댓값 | `{필드명}은(는) {N} 이하여야 합니다.` |
| 이메일 | `올바른 이메일 형식을 입력해주세요.` |
| 패턴 | `올바른 {필드명} 형식을 입력해주세요.` |

### 수정 폼 패턴 (서버 데이터 초기값)

수정 폼에서는 서버에서 조회한 데이터를 초기값으로 사용한다.

```typescript
export function ItemEditForm({ itemId }: { itemId: string }) {
  const { data: item } = useGetItemDetail(itemId);

  const form = useForm<ItemUpdateFormValues>({
    resolver: zodResolver(itemUpdateSchema),
    values: item,  // 서버 데이터가 로드되면 자동으로 폼에 반영
  });

  // ...
}
```

### 스키마 파일 위치

| 위치 | 용도 |
|------|------|
| `features/{domain}/model/{domain}Schema.ts` | 도메인별 폼 스키마 |
| `shared/lib/validations.ts` | 재사용 가능한 공통 검증 함수 |

---

## 4. URL 상태 — Next.js searchParams

### 원칙

- **필터, 페이지네이션, 탭 등 공유/북마크가 필요한 상태는 URL searchParams를 사용한다.**
- 사용자가 브라우저 뒤로가기/앞으로가기로 상태를 복원할 수 있어야 하는 경우 URL 상태를 사용한다.

### 사용 대상

| URL 상태 적합 | Zustand/로컬 상태 적합 |
|-------------|---------------------|
| 목록 페이지 필터 | 모달 열림/닫힘 |
| 페이지네이션 (page, size) | 드롭다운 열림/닫힘 |
| 정렬 (sort, order) | 임시 입력 값 |
| 탭 선택 (tab) | 폼 작성 중 임시 데이터 |
| 검색어 (keyword) | 토스트/알림 상태 |

### 사용 패턴

```typescript
// 서버 컴포넌트에서 searchParams 접근
interface PageProps {
  searchParams: Promise<{
    page?: string;
    size?: string;
    keyword?: string;
    status?: string;
  }>;
}

export default async function ItemListPage({ searchParams }: PageProps) {
  const params = await searchParams;
  const page = Number(params.page) || 1;
  const size = Number(params.size) || 20;
  const keyword = params.keyword || '';
  const status = params.status || '';

  return <ItemListContent page={page} size={size} keyword={keyword} status={status} />;
}
```

```typescript
// 클라이언트 컴포넌트에서 searchParams 변경
'use client';

import { useRouter, useSearchParams } from 'next/navigation';

export function ItemFilter() {
  const router = useRouter();
  const searchParams = useSearchParams();

  const updateFilter = (key: string, value: string) => {
    const params = new URLSearchParams(searchParams.toString());
    if (value) {
      params.set(key, value);
    } else {
      params.delete(key);
    }
    params.set('page', '1');  // 필터 변경 시 1페이지로 리셋
    router.push(`?${params.toString()}`);
  };

  return (
    // 필터 UI 렌더링
  );
}
```

### URL 상태와 TanStack Query 연동

URL searchParams를 쿼리 키에 포함시켜 필터/페이지 변경 시 자동으로 데이터를 재조회한다.

```typescript
export function useGetItemList(params: { page: number; size: number; keyword: string }) {
  return useQuery({
    queryKey: ['items', 'list', params],  // URL 파라미터가 키에 포함
    queryFn: () => getItemList(params),
  });
}
```

---

## 상태 관리 선택 의사결정 트리

새로운 상태를 추가할 때 다음 순서로 판단한다.

1. **서버에서 오는 데이터인가?**
   - 예 → **TanStack Query** (절대 Zustand에 복제하지 않는다)

2. **URL에 반영되어야 하는가?** (공유/북마크/뒤로가기 복원)
   - 예 → **Next.js searchParams**

3. **폼 입력 데이터인가?**
   - 예 → **React Hook Form + Zod**

4. **여러 컴포넌트에서 공유해야 하는 클라이언트 상태인가?**
   - 예 → **Zustand**

5. **단일 컴포넌트 내부 상태인가?**
   - 예 → **React useState**
