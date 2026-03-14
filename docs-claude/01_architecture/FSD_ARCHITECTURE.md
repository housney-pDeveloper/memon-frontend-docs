# FSD (Feature-Sliced Design) 아키텍처 가이드

## 개요

본 프로젝트는 Feature-Sliced Design(FSD) 아키텍처를 채택하고 있다. FSD는 레이어 기반의 단방향 의존성 구조를 통해 코드의 예측 가능성과 유지보수성을 극대화하는 프론트엔드 아키텍처 방법론이다.

프로젝트별로 FSD 적용 수준이 다르다:

| 프로젝트 | FSD 수준 | 설명 |
|----------|----------|------|
| **31_webadmin** | Full FSD | 5개 레이어 전체 적용 |
| **32_appuser** | Light FSD | 3개 레이어 + core 레이어 |
| **21_hs-common** | 해당 없음 | 공유 라이브러리 (플랫폼 중립 TypeScript) |
| **11_publishing** | 해당 없음 | UI 레퍼런스 |

---

## webadmin (Full FSD)

### 레이어 구조

```
src/
├── app/            ← 최상위: 앱 초기화, 프로바이더, 라우터 가드
├── widgets/        ← 레이아웃 수준 UI 블록
├── features/       ← 비즈니스 로직 + 쓰기 작업 (도메인별)
├── entities/       ← 읽기 전용 데이터 (도메인별)
└── shared/         ← 도메인 무관 공통 코드
```

의존 방향: `app → widgets → features → entities → shared`

상위 레이어는 하위 레이어만 참조할 수 있으며, 그 역방향은 허용되지 않는다.

---

### 레이어별 상세 책임

#### `app/` — 앱 초기화 및 글로벌 설정

앱 전체의 초기화, 프로바이더 구성, 라우터 가드를 담당한다.

**프로바이더 목록:**

| 프로바이더 | 역할 |
|-----------|------|
| `AuthInit` | 인증 상태 초기화 및 토큰 복원 |
| `QueryProvider` | TanStack Query 클라이언트 설정 |
| `CommonCodeProvider` | 공통 코드(코드 테이블) 로딩 |
| `NotificationInit` | 알림 시스템 초기화 |
| `SocketProvider` | WebSocket 연결 관리 |

**라우터 가드:**
- `ProtectedRoute`: 인증된 사용자만 접근 가능한 페이지 래핑
- `PermissionGuard`: 사용자 권한에 따라 컴포넌트 조건부 렌더링

---

#### `widgets/` — 레이아웃 수준 UI 블록

페이지 레이아웃을 구성하는 대규모 UI 블록을 담당한다.

```
widgets/
├── AdminLayout/     ← 관리자 전체 레이아웃
├── Header/          ← 상단 헤더
├── Sidebar/         ← 사이드 네비게이션
├── Footer/          ← 하단 푸터
└── Guards/          ← 라우터 가드 UI 래퍼
```

widgets는 features, entities, shared를 조합하여 레이아웃 수준의 UI를 구성한다.

---

#### `features/{domain}/` — 비즈니스 로직 + 쓰기 작업

도메인별 비즈니스 로직과 데이터 변경(쓰기) 작업을 담당한다.

```
features/
├── auth/
│   ├── api/        ← POST/PUT/PATCH/DELETE 요청 (쓰기 API)
│   ├── model/      ← 커스텀 훅, Zustand 스토어
│   └── ui/         ← 기능 관련 UI 컴포넌트
├── item/
│   ├── api/
│   ├── model/
│   └── ui/
└── order/
    ├── api/
    ├── model/
    └── ui/
```

| 서브디렉토리 | 역할 | 예시 |
|-------------|------|------|
| `api/` | 쓰기 전용 API (POST, PUT, PATCH, DELETE) | `createItem.ts`, `updateOrder.ts` |
| `model/` | 비즈니스 로직 훅, 상태 관리 스토어 | `useCreateItem.ts`, `useLoginForm.ts` |
| `ui/` | 비즈니스 로직이 포함된 UI 컴포넌트 | `ItemCreateForm.tsx`, `LoginDialog.tsx` |

---

#### `entities/{domain}/` — 읽기 전용 데이터

도메인별 데이터 읽기 및 표시를 담당한다. **쓰기 작업은 절대 포함하지 않는다.**

```
entities/
├── item/
│   ├── api/        ← GET 요청만 (읽기 전용 API)
│   ├── model/      ← 쿼리 훅, 타입 정의, 스토어
│   └── ui/         ← 읽기 전용 표시 컴포넌트
├── user/
│   ├── api/
│   ├── model/
│   └── ui/
└── order/
    ├── api/
    ├── model/
    └── ui/
```

| 서브디렉토리 | 역할 | 예시 |
|-------------|------|------|
| `api/` | 읽기 전용 API (GET만) | `getItemList.ts`, `getItemDetail.ts` |
| `model/` | 쿼리 훅, TypeScript 타입, 스토어 | `useGetItemList.ts`, `item.types.ts` |
| `ui/` | 데이터 표시 전용 컴포넌트 | `ItemCard.tsx`, `ItemStatusBadge.tsx` |

---

#### `shared/` — 도메인 무관 공통 코드

모든 도메인에서 공유하는 범용 코드를 담당한다.

```
shared/
├── config/         ← 앱 설정, 환경 변수
├── primitives/     ← 기본 UI 원자 (shared 내부에서만 import 가능)
├── ui/             ← 스타일 + 순수 컴포넌트 (로직 없음)
├── components/     ← 로직 포함 컴포넌트 (도메인 무관)
├── stores/         ← 글로벌 상태 스토어
├── api/            ← API 클라이언트, 인터셉터
├── lib/            ← 유틸리티 함수
├── hooks/          ← 범용 커스텀 훅
├── types/          ← 공통 타입 정의
└── constants/      ← 상수 값
```

| 서브디렉토리 | 역할 | 예시 |
|-------------|------|------|
| `config/` | 앱 설정, 환경 변수 관리 | `env.ts`, `site.ts` |
| `primitives/` | 기본 UI 원자 요소 | Radix UI 래퍼, 아이콘 |
| `ui/` | 순수 표시 컴포넌트 (비즈니스 로직 없음) | `Button`, `Input`, `Badge` |
| `components/` | 로직 포함 범용 컴포넌트 | `DataGrid`, `AutoComplete`, `TreeSearch` |
| `stores/` | 글로벌 Zustand 스토어 | `useThemeStore`, `useSidebarStore` |
| `api/` | Axios 인스턴스, 인터셉터 | `apiClient.ts`, `interceptors.ts` |
| `lib/` | 유틸리티 함수 | `formatDate.ts`, `cn.ts` |
| `hooks/` | 범용 훅 | `useDebounce.ts`, `useMediaQuery.ts` |
| `types/` | 공통 타입 정의 | `api.types.ts`, `common.types.ts` |
| `constants/` | 상수 값 | `routes.ts`, `queryKeys.ts` |

---

### 의존성 규칙 (ESLint 강제)

#### 1. 단방향 의존성

상위 레이어는 하위 레이어만 참조할 수 있다. 역방향 참조는 금지된다.

```
[허용]
app       → widgets, features, entities, shared
widgets   → features, entities, shared
features  → entities, shared
entities  → shared

[금지]
shared    → entities, features, widgets, app
entities  → features, widgets, app
features  → widgets, app
widgets   → app
```

#### 2. 크로스 도메인 import 금지

같은 레이어 내에서 다른 도메인의 코드를 직접 import할 수 없다.

```typescript
// [금지] entities/item/ 에서 entities/order/ 를 import
import { OrderType } from '@/entities/order/model/order.types';

// [금지] features/item/ 에서 features/order/ 를 import
import { useCreateOrder } from '@/features/order/model/useCreateOrder';
```

도메인 간 데이터가 필요한 경우, 상위 레이어(widgets 또는 app)에서 조합한다.

#### 3. shared의 상향 참조 금지

shared는 entities, features, widgets를 절대 import할 수 없다.

```typescript
// [금지] shared/api/apiClient.ts 에서
import { useAuthStore } from '@/entities/user/model/store';
```

#### 4. 배럴 import 금지

레이어 루트의 배럴 파일(`features/index.ts`, `entities/index.ts`)을 통한 import는 금지된다. 항상 직접 경로로 import한다.

```typescript
// [금지] 배럴 import
import { useLogin } from '@/features';
import { useLogin } from '@/features/auth';

// [허용] 직접 경로 import
import { useLogin } from '@/features/auth/model/useLogin';
```

#### 5. primitives 접근 제한

`shared/primitives/`는 `shared/` 내부에서만 import할 수 있다. 외부 레이어(entities, features 등)에서 직접 import할 수 없다.

```typescript
// [금지] features/auth/ui/LoginForm.tsx 에서
import { Checkbox } from '@/shared/primitives/checkbox';

// [허용] shared/ui/Checkbox.tsx 에서
import { Checkbox as PrimitiveCheckbox } from '@/shared/primitives/checkbox';
```

---

### 콜백 패턴 (FSD 규칙 준수를 위한 의존성 역전)

shared는 entities/features를 import할 수 없으므로, 상위 레이어에서 콜백을 주입하는 패턴을 사용한다.

**대표 사례: API 클라이언트의 토큰 갱신**

shared의 API 클라이언트는 인증 토큰이 필요하지만, 토큰 관리는 entities/features 영역이다. 이를 해결하기 위해 콜백 등록 방식을 사용한다.

```typescript
// shared/api/apiClient.ts — 콜백 슬롯만 정의
let onTokenRefresh: (() => Promise<string>) | null = null;

export function registerTokenRefreshCallback(cb: () => Promise<string>) {
  onTokenRefresh = cb;
}

// 인터셉터에서 콜백 호출
axiosInstance.interceptors.response.use(null, async (error) => {
  if (error.response?.status === 401 && onTokenRefresh) {
    const newToken = await onTokenRefresh();
    // 토큰 갱신 후 재시도
  }
});
```

```typescript
// app/api/setupApiClient.ts — 앱 초기화 시 콜백 등록
import { registerTokenRefreshCallback } from '@/shared/api/apiClient';
import { refreshToken } from '@/features/auth/api/refreshToken';

registerTokenRefreshCallback(async () => {
  const result = await refreshToken();
  return result.accessToken;
});
```

이 패턴으로 shared는 상위 레이어를 알지 못한 채 기능을 확장할 수 있다.

---

### UI 컴포넌트 분류 체계

UI 컴포넌트는 위치에 따라 역할과 제약이 명확히 구분된다.

| 위치 | 역할 | 로직 포함 | 도메인 의존 | 예시 |
|------|------|----------|------------|------|
| `shared/ui/` | 스타일 + 순수 컴포넌트 | 불가 | 불가 | `Button`, `Input`, `Badge`, `Dialog` |
| `shared/components/` | 로직 포함 범용 컴포넌트 | 가능 (도메인 무관) | 불가 | `DataGrid`, `AutoComplete`, `TreeSearch` |
| `entities/*/ui/` | 읽기 전용 표시 컴포넌트 | 읽기 전용 | 특정 도메인 | `ItemCard`, `UserAvatar`, `OrderStatusBadge` |
| `features/*/ui/` | 비즈니스 로직 컴포넌트 | 가능 | 특정 도메인 | `ItemCreateForm`, `LoginDialog`, `OrderApprovalButton` |
| `widgets/` | 레이아웃 수준 조합 | 조합 로직 | 다중 도메인 | `AdminLayout`, `Header`, `Sidebar` |

**판단 기준:**

1. 도메인에 의존하는가?
   - 아니오 → `shared/ui/` 또는 `shared/components/`
   - 예 → 2번으로
2. 데이터를 변경(쓰기)하는가?
   - 아니오 (읽기/표시만) → `entities/*/ui/`
   - 예 → `features/*/ui/`
3. 여러 도메인을 조합하는가?
   - 예 → `widgets/`

---

## appuser (Light FSD)

### 레이어 구조

```
src/
├── app/            ← 앱 초기화, 라우팅
├── core/           ← 플랫폼 추상화 (v2 RN 마이그레이션 핵심)
├── features/       ← 도메인별 비즈니스 로직 + UI
└── shared/         ← 공통 코드
```

의존 방향: `app → features → shared`, `core`는 shared와 동일 레벨

appuser는 entities 레이어와 widgets 레이어를 사용하지 않는다. 읽기/쓰기 구분 없이 features에서 통합 관리한다.

---

### `core/` — 플랫폼 추상화 레이어

**v2 React Native 마이그레이션의 핵심 레이어이다.**

현재(v1)는 Capacitor 기반 웹앱이며, v2에서 React Native로 전환 예정이다. `core/` 레이어는 플랫폼 의존 코드를 추상화하여 마이그레이션 영향 범위를 최소화한다.

#### `core/storage.ts` — 스토리지 추상화

```typescript
// 인터페이스 정의
export interface IStorage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
  removeItem(key: string): Promise<void>;
}

// v1 (현재): 웹 구현체
export class WebStorage implements IStorage {
  async getItem(key: string) { return localStorage.getItem(key); }
  async setItem(key: string, value: string) { localStorage.setItem(key, value); }
  async removeItem(key: string) { localStorage.removeItem(key); }
}

// v2 (예정): React Native 구현체
// export class NativeStorage implements IStorage {
//   async getItem(key: string) { return AsyncStorage.getItem(key); }
//   ...
// }
```

#### 핵심 규칙

- **모든 플랫폼 의존 코드는 반드시 `core/`를 통해야 한다.**
- **`localStorage`, `window`, `document` 등의 브라우저 API 직접 접근은 금지된다.**
- v2 마이그레이션 시 `core/` 내부 구현체만 교체하면 나머지 코드는 변경 없이 동작해야 한다.

```typescript
// [금지] features 내에서 직접 브라우저 API 사용
const token = localStorage.getItem('token');

// [허용] core를 통한 스토리지 접근
import { storage } from '@/core/storage';
const token = await storage.getItem('token');
```

---

### features/{domain}/ 구조

appuser의 features는 webadmin과 달리 별도의 `api/` 디렉토리를 두지 않는 것이 일반적이다.

```
features/
├── auth/
│   ├── ui/         ← UI 컴포넌트
│   └── model/      ← 훅, 스토어, API 호출 통합
├── home/
│   ├── ui/
│   └── model/
└── mypage/
    ├── ui/
    └── model/
```

---

### shared/ 구조

```
shared/
├── ui/             ← 순수 Tailwind 컴포넌트 (Radix UI 사용 금지)
├── api/            ← API 클라이언트
├── hooks/          ← 범용 커스텀 훅
├── lib/            ← 유틸리티 함수
└── config/         ← 설정
```

**중요:** appuser의 shared/ui/는 **순수 Tailwind CSS로만 구성**한다. Radix UI, shadcn/ui 등의 외부 컴포넌트 라이브러리를 사용하지 않는다. 이는 v2 React Native 전환 시 호환성을 확보하기 위함이다.

---

## 새 도메인 추가 가이드

### webadmin에서 새 도메인 추가

entities와 features 양쪽 모두에 디렉토리를 생성한다.

```bash
# 1. entities 디렉토리 생성 (읽기 전용 데이터)
mkdir -p src/entities/{domain}/api
mkdir -p src/entities/{domain}/model
mkdir -p src/entities/{domain}/ui

# 2. features 디렉토리 생성 (비즈니스 로직 + 쓰기)
mkdir -p src/features/{domain}/api
mkdir -p src/features/{domain}/model
mkdir -p src/features/{domain}/ui
```

**최소 파일 구성:**

```
entities/{domain}/
├── api/
│   └── get{Domain}List.ts      ← GET 요청
├── model/
│   ├── {domain}.types.ts       ← 타입 정의
│   └── useGet{Domain}List.ts   ← TanStack Query 훅
└── ui/
    └── {Domain}Card.tsx         ← 읽기 전용 표시 컴포넌트

features/{domain}/
├── api/
│   ├── create{Domain}.ts       ← POST 요청
│   ├── update{Domain}.ts       ← PUT/PATCH 요청
│   └── delete{Domain}.ts       ← DELETE 요청
├── model/
│   ├── useCreate{Domain}.ts    ← 생성 뮤테이션 훅
│   ├── useUpdate{Domain}.ts    ← 수정 뮤테이션 훅
│   └── useDelete{Domain}.ts    ← 삭제 뮤테이션 훅
└── ui/
    └── {Domain}CreateForm.tsx   ← 폼 UI
```

### appuser에서 새 도메인 추가

features에만 디렉토리를 생성한다.

```bash
mkdir -p src/features/{domain}/model
mkdir -p src/features/{domain}/ui
```

**최소 파일 구성:**

```
features/{domain}/
├── model/
│   ├── {domain}.types.ts       ← 타입 정의
│   ├── useGet{Domain}List.ts   ← 데이터 조회 훅
│   └── use{Domain}Actions.ts   ← 데이터 변경 훅
└── ui/
    ├── {Domain}Screen.tsx       ← 메인 화면
    └── {Domain}Card.tsx         ← 카드 컴포넌트
```

---

## 요약 비교표

| 항목 | webadmin (Full FSD) | appuser (Light FSD) |
|------|-------------------|-------------------|
| 레이어 수 | 5개 (app, widgets, features, entities, shared) | 3개 + core (app, features, shared + core) |
| entities 레이어 | 있음 (읽기 전용 분리) | 없음 |
| widgets 레이어 | 있음 (레이아웃 조합) | 없음 |
| core 레이어 | 없음 | 있음 (플랫폼 추상화) |
| UI 라이브러리 | Radix UI + shadcn/ui | 순수 Tailwind CSS |
| API 분리 | entities (GET) / features (POST/PUT/DELETE) | features 내 통합 |
| 기술 스택 | Next.js 15 | Next.js 15 (v1), React Native (v2 예정) |
