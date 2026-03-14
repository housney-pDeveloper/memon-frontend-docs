# 라우팅 가이드

## 개요

본 프로젝트는 Next.js 15의 App Router를 사용한다. 인증과 권한 검증은 3단계 가드 계층으로 처리된다.

| 프로젝트 | 포트 | 설명 |
|----------|------|------|
| **31_webadmin** | 8001 | 관리자 대시보드 |
| **32_appuser** | 8002 | 사용자 앱 |

---

## 1. Next.js App Router

### 파일 기반 라우팅

두 프로젝트 모두 `src/app/` 디렉토리 내에서 파일 기반 라우팅을 사용한다.

```
src/app/
├── layout.tsx          ← 루트 레이아웃
├── page.tsx            ← / (루트 페이지, 로그인)
├── dashboard/
│   └── page.tsx        ← /dashboard
├── items/
│   ├── page.tsx        ← /items (목록)
│   └── [id]/
│       └── page.tsx    ← /items/:id (상세)
└── (admin)/            ← 라우트 그룹 (URL에 반영되지 않음)
    ├── layout.tsx      ← 관리자 전용 레이아웃
    └── settings/
        └── page.tsx    ← /settings
```

### 주요 파일 컨벤션

| 파일 | 역할 |
|------|------|
| `page.tsx` | 라우트의 UI |
| `layout.tsx` | 공유 레이아웃 (하위 라우트에 적용) |
| `loading.tsx` | 로딩 UI (Suspense 경계) |
| `error.tsx` | 에러 UI (Error Boundary) |
| `not-found.tsx` | 404 UI |

---

## 2. 미들웨어 (`src/middleware.ts`)

서버 사이드에서 실행되며, 모든 요청에 대해 인증 상태를 확인하고 라우팅을 제어한다.

### 동작 흐름

```
요청 수신
  │
  ├── matcher 제외 경로? ──→ 통과 (미들웨어 미적용)
  │
  ├── 개발 모드? ──→ 통과 (미들웨어 바이패스)
  │
  ├── 인증 쿠키 확인 (auth-storage)
  │   │
  │   ├── 미인증 + 보호된 경로 ──→ / (로그인 페이지)로 리다이렉트
  │   │
  │   └── 인증됨 + / (루트) 접근 ──→ /dashboard로 리다이렉트
  │
  └── 그 외 ──→ 통과
```

### 인증 확인 방식

미들웨어는 `auth-storage` 쿠키를 통해 인증 상태를 확인한다. 이 쿠키는 Zustand persist 미들웨어에 의해 관리된다.

```typescript
// src/middleware.ts
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
  // 개발 모드에서는 미들웨어 바이패스
  if (process.env.NODE_ENV === 'development') {
    return NextResponse.next();
  }

  const authCookie = request.cookies.get('auth-storage');
  const isAuthenticated = authCookie ? checkAuthState(authCookie.value) : false;
  const { pathname } = request.nextUrl;

  // 미인증 사용자 → 로그인 페이지로 리다이렉트
  if (!isAuthenticated && pathname !== '/') {
    return NextResponse.redirect(new URL('/', request.url));
  }

  // 인증된 사용자가 루트(/) 접근 시 → 대시보드로 리다이렉트
  if (isAuthenticated && pathname === '/') {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}
```

### Matcher 설정

미들웨어가 적용되지 않는 경로를 정의한다. 정적 자원과 API 경로는 미들웨어를 거치지 않는다.

```typescript
export const config = {
  matcher: [
    /*
     * 다음 경로를 제외한 모든 요청에 미들웨어 적용:
     * - _next/static (정적 파일)
     * - _next/image (이미지 최적화)
     * - favicon.ico (파비콘)
     * - fonts (폰트 파일)
     * - images (이미지 파일)
     * - api (API 라우트)
     */
    '/((?!_next/static|_next/image|favicon.ico|fonts|images|api).*)',
  ],
};
```

### 개발 모드 바이패스

개발 환경(`NODE_ENV === 'development'`)에서는 미들웨어가 바이패스된다. 이를 통해 개발 시 인증 없이 모든 페이지에 접근할 수 있다.

**주의:** 라우트 가드 로직을 테스트하려면 프로덕션 빌드(`next build && next start`)로 확인해야 한다.

---

## 3. 라우트 가드

클라이언트 사이드에서 실행되며, 미들웨어를 보완하는 2단계 보호 계층을 제공한다.

### 가드 계층 구조

```
[1단계] Middleware (서버)
  │  쿠키 기반 인증 확인
  │  미인증 → 로그인 리다이렉트
  │
  ▼
[2단계] ProtectedRoute (클라이언트)
  │  Zustand 스토어 기반 인증 확인
  │  미인증 → 로그인 리다이렉트
  │  토큰 만료 감지 → 재인증
  │
  ▼
[3단계] PermissionGuard (컴포넌트)
     사용자 권한에 따라 UI 조건부 렌더링
     권한 없음 → 접근 불가 UI 또는 숨김
```

### `getAuthStateSync()` — 동기 인증 상태 확인

`app/router/guards.ts`에 정의된 동기 함수로, Zustand 스토어에서 인증 상태를 즉시 확인한다.

```typescript
// app/router/guards.ts
import { useAuthStore } from '@/entities/auth/model/store';

/**
 * 동기적으로 인증 상태를 확인한다.
 * Zustand 스토어의 getState()를 사용하여 React 컴포넌트 외부에서도 호출 가능.
 */
export function getAuthStateSync(): boolean {
  const state = useAuthStore.getState();
  return state.isAuthenticated;
}
```

### `ProtectedRoute` — 인증 필수 페이지 래퍼

인증이 필요한 페이지를 감싸는 컴포넌트이다. 인증되지 않은 사용자는 로그인 페이지로 리다이렉트된다.

```typescript
// 사용 예시: 레이아웃에서 적용
// src/app/(admin)/layout.tsx
import { ProtectedRoute } from '@/widgets/Guards/ProtectedRoute';

export default function AdminLayout({ children }: { children: React.ReactNode }) {
  return (
    <ProtectedRoute>
      <AdminLayout>
        {children}
      </AdminLayout>
    </ProtectedRoute>
  );
}
```

**ProtectedRoute의 역할:**
- Zustand 스토어에서 인증 상태 확인
- 미인증 시 로그인 페이지(`/`)로 리다이렉트
- 토큰 만료 감지 시 재인증 프로세스 트리거
- 인증 확인 중 로딩 UI 표시

### `PermissionGuard` — 권한 기반 조건부 렌더링

사용자 권한에 따라 특정 UI 요소를 조건부로 렌더링한다.

```typescript
// 사용 예시: 특정 권한이 필요한 버튼
<PermissionGuard permission="item:create">
  <Button onClick={handleCreate}>상품 등록</Button>
</PermissionGuard>

// 권한 없을 때 대체 UI 표시
<PermissionGuard
  permission="item:delete"
  fallback={<span>삭제 권한이 없습니다</span>}
>
  <Button onClick={handleDelete}>삭제</Button>
</PermissionGuard>
```

### 가드 적용 범위

| 가드 | 적용 위치 | 확인 대상 | 실패 시 동작 |
|------|----------|----------|------------|
| Middleware | 서버 (모든 요청) | 쿠키 (`auth-storage`) | 로그인 페이지 리다이렉트 |
| ProtectedRoute | 클라이언트 (레이아웃) | Zustand 스토어 | 로그인 페이지 리다이렉트 |
| PermissionGuard | 클라이언트 (컴포넌트) | 사용자 권한 | 숨김 또는 대체 UI |

**왜 미들웨어와 ProtectedRoute를 모두 사용하는가?**

- **미들웨어 (서버):** 초기 페이지 로드 시 서버에서 빠르게 리다이렉트한다. 그러나 쿠키 기반이므로 토큰 만료 등 세밀한 상태를 확인하기 어렵다.
- **ProtectedRoute (클라이언트):** SPA 내비게이션 시 클라이언트에서 인증 상태를 확인한다. 토큰 만료 감지, 재인증 등 세밀한 처리가 가능하다.
- 두 계층을 조합하여 서버/클라이언트 양쪽에서 빈틈없는 인증 보호를 구현한다.

---

## 4. API 리라이트 (next.config.mjs)

Next.js의 rewrites 설정을 통해 프론트엔드의 `/api/*` 요청을 백엔드 API 서버로 프록시한다.

### 설정

```javascript
// next.config.mjs
const API_BASE_URL = process.env.API_BASE_URL || 'http://localhost:9999';

/** @type {import('next').NextConfig} */
const nextConfig = {
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: `${API_BASE_URL}/api/:path*`,
      },
    ];
  },
};

export default nextConfig;
```

### 동작 방식

```
브라우저 요청: GET http://localhost:8001/api/items
                          │
                          ▼
Next.js 서버:    rewrites 규칙 매칭
                          │
                          ▼
백엔드 프록시:   GET http://localhost:9999/api/items
```

### 프록시의 목적

| 목적 | 설명 |
|------|------|
| CORS 회피 | 같은 도메인(localhost:8001)에서 API를 호출하므로 CORS 문제가 발생하지 않는다 |
| 보안 | 백엔드 서버의 실제 주소가 브라우저에 노출되지 않는다 |
| 환경 분리 | `API_BASE_URL` 환경 변수만 변경하여 개발/스테이징/프로덕션 서버를 전환할 수 있다 |

### 환경별 API_BASE_URL 설정

```bash
# .env.local (로컬 개발)
API_BASE_URL=http://localhost:9999

# .env.staging (스테이징)
API_BASE_URL=https://api-staging.example.com

# .env.production (프로덕션)
API_BASE_URL=https://api.example.com
```

### 주의사항

- Next.js의 API Routes(`src/app/api/`)와 혼동하지 않는다. 본 프로젝트에서 `/api/*`는 백엔드 프록시 전용이다.
- 미들웨어 matcher에서 `api`가 제외되어 있으므로, API 요청은 인증 미들웨어를 거치지 않는다. 인증은 백엔드에서 처리한다.
- 프록시는 서버 사이드에서만 동작한다. 클라이언트에서는 `/api/*` 경로로 요청하면 Next.js 서버가 자동으로 프록시한다.
