# 인증/인가 가이드

> webadmin(관리자 대시보드)과 appuser(모바일 사용자 앱) 공통 인증/인가 아키텍처 가이드.
> 두 프로젝트 모두 Next.js 15 기반이며, FSD(Feature-Sliced Design) 아키텍처를 따른다.

---

## 1. 토큰 관리 전략

### 1.1 Access Token — 메모리 전용

Access Token은 **JavaScript 변수(메모리)에만 저장**한다. localStorage, sessionStorage, cookie 등 영속 스토리지에 절대 저장하지 않는다.

```
[원칙] Access Token은 XSS 공격 시 탈취 가능한 스토리지에 저장하지 않는다.
       메모리에만 유지하면 페이지 새로고침 시 사라지지만,
       Refresh Token으로 재발급받는 구조로 보완한다.
```

**API** (`shared/lib/authStorage.ts`):

| 함수 | 설명 |
|---|---|
| `getAccessToken()` | 메모리에 저장된 Access Token 반환 |
| `setAccessToken(token)` | Access Token을 메모리 변수에 저장 |

- 내부적으로 모듈 스코프 변수 `memoryAccessToken`에 보관
- 새로고침(F5) 시 토큰이 사라지므로, `/auth/refresh` API를 통해 재발급받는다

### 1.2 Refresh Token — Zustand Persist

Refresh Token은 Zustand의 `persist` 미들웨어를 통해 관리한다. 사용자의 `rememberMe` 선택에 따라 스토리지가 달라진다.

| 조건 | 스토리지 | 동작 |
|---|---|---|
| `rememberMe = true` | `localStorage` | 브라우저를 닫아도 유지 |
| `rememberMe = false` | `sessionStorage` | 탭/브라우저 종료 시 삭제 |

- **Zustand persist key**: `auth-storage`
- Zustand persist의 `storage` 옵션에서 `rememberMe` 값에 따라 `localStorage` 또는 `sessionStorage`를 동적으로 선택한다

### 1.3 Remember Me / 저장된 이메일

| 키 | 스토리지 | 값 | 설명 |
|---|---|---|---|
| `auth-remember` | `localStorage` | `'1'` 또는 키 자체 삭제 | Remember me 체크 여부 |
| `auth-saved-email` | `localStorage` | 이메일 문자열 | 로그인 폼에 미리 채울 이메일 (JWT 아님) |

```
[주의] auth-saved-email에는 이메일 주소만 저장한다.
       토큰이나 비밀번호 등 민감 정보를 이 키에 저장하지 않는다.
```

---

## 2. RSA 암호화

### 2.1 개요

로그인 시 비밀번호를 **RSA-OAEP(SHA-256)** 알고리즘으로 암호화하여 전송한다. 평문 비밀번호가 네트워크를 통해 전달되지 않도록 보호한다.

**구현 위치**: `shared/api/crypto.ts`

### 2.2 공개키 관리

| 항목 | 설명 |
|---|---|
| 조회 API | `GET /auth/getPublicKey` |
| 1차 캐시 | 모듈 스코프 메모리 변수 |
| 2차 캐시 | `sessionStorage` (키: `rsa-public-key`) |
| 캐시 전략 | 메모리 확인 → sessionStorage 확인 → 서버 조회 순 |

### 2.3 주요 함수

| 함수 | 설명 |
|---|---|
| `ensurePublicKey()` | 공개키가 캐시에 없으면 서버에서 조회하여 캐시에 저장 |
| `encryptWithPublicKey(plainText)` | 주어진 평문을 RSA-OAEP로 암호화하여 Base64 문자열 반환 |

### 2.4 기술 요구사항

- **Web Crypto API** 사용: `window.crypto.subtle`
- Web Crypto API는 **Secure Context**(HTTPS 또는 localhost)에서만 동작한다
- 개발 환경에서 HTTP로 접근 시 `localhost`를 사용해야 한다 (IP 접근 불가)

```
[중요] Web Crypto API는 http://192.168.x.x 같은 비보안 컨텍스트에서 동작하지 않는다.
       개발 시 반드시 https:// 또는 http://localhost를 사용할 것.
```

---

## 3. 인증 플로우

### 3.1 로그인 플로우

```
사용자 입력 (이메일 + 비밀번호)
        |
        v
[1] ensurePublicKey() — RSA 공개키 확보
        |
        v
[2] encryptWithPublicKey(password) — 비밀번호 RSA 암호화
        |
        v
[3] POST /login { email, encryptedPassword }
        |
        v
[4] 응답 수신: { accessToken, refreshToken, ... }
        |
        v
[5] setAccessToken(accessToken) — 메모리에 저장
        |
        v
[6] Zustand auth store에 refreshToken 저장 (persist → 스토리지 자동 반영)
        |
        v
[7] rememberMe에 따라 auth-remember, auth-saved-email 처리
```

### 3.2 Silent Refresh (자동 갱신)

앱 초기 로드 시 `AuthInit` provider가 다음을 수행한다:

1. Zustand persist에서 Refresh Token 복원
2. Refresh Token이 존재하면 `/auth/refresh` API 호출
3. 새 Access Token을 메모리에 저장
4. 실패 시 로그아웃 처리

```
[AuthInit Provider]
        |
        v
  Zustand persist rehydrate 완료
        |
        v
  refreshToken 존재?
    ├── Yes → POST /auth/refresh → 새 accessToken 메모리 저장
    └── No  → 미인증 상태 유지 (로그인 페이지로 리다이렉트)
```

### 3.3 Token Refresh (401 자동 재시도)

API 요청 중 401 Unauthorized 또는 JWT 관련 에러가 발생하면 자동으로 토큰 리프레시를 시도한다.

**동시 요청 처리 패턴**:

```
요청 A (401 발생)
        |
        v
  isRefreshing === false?
    ├── Yes → isRefreshing = true
    |         POST /auth/refresh
    |              |
    |              v
    |         성공? ──Yes──> 새 토큰 저장
    |           |            failedQueue 일괄 재시도
    |           |            isRefreshing = false
    |           |
    |           └── No ──> Alert("세션 만료")
    |                       failedQueue 일괄 reject
    |                       clearAllAuthStorage()
    |                       window.location.href = '/login' (full reload)
    |
    └── No  → failedQueue에 요청 추가 (Promise 대기)
              리프레시 완료 시 자동 재시도됨
```

**핵심 변수**:

| 변수 | 타입 | 설명 |
|---|---|---|
| `isRefreshing` | `boolean` | 현재 리프레시 진행 중 여부 |
| `failedQueue` | `Array<{ resolve, reject }>` | 리프레시 대기 중인 요청 큐 |

```
[주의] 리프레시 실패 시 full reload(window.location.href)를 수행한다.
       SPA 내부 라우팅(router.push)이 아닌 전체 페이지 리로드로
       메모리의 모든 인증 상태를 확실히 초기화한다.
```

### 3.4 로그아웃

`clearAllAuthStorage()` 호출 시 다음이 모두 정리된다:

| 대상 | 처리 |
|---|---|
| 메모리 Access Token | `setAccessToken(null)` |
| localStorage | `auth-storage`, `auth-remember` 등 제거 |
| sessionStorage | `auth-storage`, `rsa-public-key` 등 제거 |
| Zustand store | 초기 상태로 리셋 |

---

## 4. 라우트 보호

### 4.1 Server-Side: `middleware.ts`

Next.js 미들웨어에서 cookie 기반으로 인증 여부를 체크한다.

| 상태 | 동작 |
|---|---|
| 미인증 + 보호 경로 접근 | 로그인 페이지로 리다이렉트 |
| 인증 + 로그인 페이지 접근 | 메인 페이지로 리다이렉트 |

### 4.2 Client-Side: `ProtectedRoute`

클라이언트에서 인증 상태와 조직 선택 상태를 확인한다.

| 상태 | 리다이렉트 대상 |
|---|---|
| 미인증 | `/` (로그인 페이지) |
| 인증 + 조직 미선택 | `/auth/select-org` |
| 인증 + 조직 선택 완료 | 요청한 페이지 표시 |

### 4.3 Component-Level: `PermissionGuard`

권한(permission) 기반으로 UI 요소의 표시/숨김을 제어한다.

```tsx
// 예시: ADMIN 권한이 있을 때만 버튼 표시
<PermissionGuard permission="ADMIN">
  <DeleteButton />
</PermissionGuard>
```

- 권한이 없으면 children을 렌더링하지 않는다
- 페이지 단위가 아닌 컴포넌트 단위의 세밀한 권한 제어에 사용

---

## 5. FSD 아키텍처 준수

### 5.1 레이어 의존성 규칙

FSD에서 `shared` 레이어는 `entities` 레이어를 import할 수 없다. 그러나 API 인터셉터(`shared/api`)에서 토큰 갱신 시 auth store(`entities/auth`)를 업데이트해야 하는 상황이 발생한다.

### 5.2 콜백 주입 패턴

이 문제를 **콜백 주입**으로 해결한다.

**설정 위치**: `app/api/setupApiClient.ts`

```
[app 레이어] setupApiClient.ts
        |
        v
  setApiClientAuthCallbacks({
    onTokensRefreshed: (tokens) => {
      // entities/auth store 업데이트
      authStore.getState().setTokens(tokens)
    }
  })
        |
        v
[shared 레이어] apiClient.ts
  - 콜백을 모듈 변수로 보관
  - 인터셉터에서 토큰 리프레시 성공 시 콜백 호출
  - shared는 entities를 직접 import하지 않음
```

**함수 시그니처**:

```typescript
setApiClientAuthCallbacks({
  onTokensRefreshed: (tokens: { accessToken: string; refreshToken: string }) => void
})
```

```
[원칙] shared → entities 직접 import 금지.
       app 레이어에서 콜백을 주입하여 레이어 간 의존성 규칙을 준수한다.
```

---

## 6. appuser 차이점

appuser는 향후 React Native(v2) 전환을 대비하여 스토리지 접근을 추상화한다.

### 6.1 IStorage 인터페이스

**위치**: `core/storage.ts`

```typescript
interface IStorage {
  getItem(key: string): string | null | Promise<string | null>
  setItem(key: string, value: string): void | Promise<void>
  removeItem(key: string): void | Promise<void>
}
```

### 6.2 토큰 접근 방식

| 프로젝트 | 접근 방식 |
|---|---|
| **webadmin** | `getAccessToken()` / `setAccessToken()` (모듈 직접 호출) |
| **appuser** | `authStorage.getToken()` / `authStorage.setToken()` (`IStorage` 경유) |

### 6.3 v2 전환 전략

```
[현재 - Next.js (v1)]
  IStorage 구현체 = Web Storage API (localStorage/sessionStorage)

[향후 - React Native (v2)]
  IStorage 구현체 = AsyncStorage 또는 SecureStore
  → 인터페이스가 동일하므로 구현체만 교체하면 된다
```

```
[핵심] appuser에서 스토리지 접근 시 반드시 IStorage 인터페이스를 통해 접근한다.
       localStorage/sessionStorage를 직접 호출하면 v2 전환 시 모두 수정해야 한다.
```

---

## 요약: 토큰 저장 위치 종합

| 데이터 | 저장 위치 | 수명 |
|---|---|---|
| Access Token | 메모리 (JS 변수) | 페이지 새로고침 시 소멸 |
| Refresh Token | Zustand persist → localStorage 또는 sessionStorage | rememberMe에 따라 결정 |
| RSA 공개키 | 메모리 + sessionStorage | 탭 종료 시 소멸 |
| Remember Me 플래그 | localStorage (`auth-remember`) | 수동 삭제 전까지 유지 |
| 저장된 이메일 | localStorage (`auth-saved-email`) | 수동 삭제 전까지 유지 |
