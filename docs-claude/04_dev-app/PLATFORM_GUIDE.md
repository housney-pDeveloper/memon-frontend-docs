# 32_appuser 플랫폼 추상화 및 모바일 전환 가이드

> Capacitor v1 (현재) → React Native v2 (계획) 전환을 위한 아키텍처 가이드

---

## 1. core/ 레이어 개요

플랫폼 의존적인 코드를 추상화하는 계층이다. v2(React Native) 전환 시 이 폴더만 교체하면 나머지 코드 수정이 불필요하다.

현재 구조:

```
core/
├── storage.ts   # IStorage 인터페이스 + WebStorage 구현
└── index.ts     # barrel export
```

---

## 2. IStorage 인터페이스

```typescript
export interface IStorage {
  getItem(key: string): Promise<string | null>
  setItem(key: string, value: string): Promise<void>
  removeItem(key: string): Promise<void>
  clear(): Promise<void>
}
```

### 버전별 구현체

- **v1**: `WebStorage` — localStorage를 Promise로 래핑
- **v2**: `RNStorage` — AsyncStorage/SecureStore 기반

### authStorage 헬퍼

| 메서드 | 설명 |
|--------|------|
| `getToken()` | 액세스 토큰 조회 |
| `setToken()` | 액세스 토큰 저장 |
| `getRefreshToken()` | 리프레시 토큰 조회 |
| `setRefreshToken()` | 리프레시 토큰 저장 |
| `clearAll()` | 모든 인증 정보 삭제 |

---

## 3. v2 전환 시 재사용 가능 영역

| 재사용 가능 (80~90%) | 재작성 필요 |
|----------------------|-------------|
| `core/` (인터페이스 동일, 구현만 교체) | `shared/ui/` (RN 컴포넌트) |
| `hooks/` (비즈니스 로직) | `routes/` (React Navigation) |
| `stores/` (Zustand) | 네이티브 모듈 |
| `types/` (타입 정의) | |
| `api/client.ts` (Axios) | |
| `@hs/common` (공유 라이브러리) | |

---

## 4. 새 플랫폼 기능 추가 시 규칙

플랫폼 API(카메라, GPS, 푸시 등) 사용 시 반드시 `core/`에 인터페이스를 정의한다.

```typescript
// core/camera.ts — 인터페이스 정의
export interface ICamera {
  takePhoto(): Promise<PhotoResult>
  pickFromGallery(): Promise<PhotoResult>
}

// core/camera.web.ts — v1 Web 구현체
export class WebCamera implements ICamera {
  async takePhoto(): Promise<PhotoResult> {
    // Capacitor Camera 플러그인 사용
  }
}

// core/camera.native.ts — v2 RN 구현체 (전환 시 작성)
export class NativeCamera implements ICamera {
  async takePhoto(): Promise<PhotoResult> {
    // react-native-camera 사용
  }
}
```

- 구현체는 v1(Web), v2(RN) 각각 작성
- `features/`나 `shared/`에서는 `core/` 인터페이스만 import

---

## 5. 모바일 전환 로드맵

### v1 (현재): Capacitor + WebView

- 빠른 출시 목적
- React + Tailwind CSS 기반 웹앱을 WebView로 래핑
- 플랫폼 API는 Capacitor 플러그인으로 접근

### v2 (계획): React Native

- 네이티브 성능 최적화
- `core/` 폴더의 구현체만 교체
- UI는 React Native 컴포넌트로 재작성
- 비즈니스 로직(hooks, stores, api)은 그대로 재사용

---

## 6. 모바일 UX 가이드라인

### 터치 타겟

최소 44x44px 크기를 보장한다.

```tsx
<button className="min-h-[44px] min-w-[44px] ...">
  탭
</button>
```

### 폼 입력

모바일 키보드 대응을 위해 `inputMode` 속성을 활용한다.

```tsx
<input inputMode="email" />    // 이메일 키보드
<input inputMode="tel" />      // 숫자 키보드
<input inputMode="numeric" />  // 숫자 전용
```

### 스크롤

자연스러운 모멘텀 스크롤을 적용한다.

```css
-webkit-overflow-scrolling: touch;
```

### 이미지

- 지연 로딩 적용 (`loading="lazy"`)
- 적절한 크기의 이미지 사용 (불필요한 대용량 이미지 금지)

### 네트워크

- 로딩 상태를 항상 표시
- 오프라인 상황에 대비한 에러 처리

### 폰트 크기

- 최소 14px 유지 (가독성 확보)

### 여백

하단 safe area를 대응한다.

```css
padding-bottom: env(safe-area-inset-bottom);
```
