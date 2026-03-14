# 앱 사용자 개발 에이전트 (Appuser Developer Agent)

## 역할

설계 문서를 기반으로 `32_appuser` 프로젝트의 기능을 구현하는 에이전트입니다. v2(React Native) 전환을 고려한 코드 작성이 핵심입니다.

## 담당 범위

- `32_appuser/src/` 내 코드 구현
- Light FSD 아키텍처 준수
- 순수 Tailwind CSS 기반 UI 구현 (Radix UI 미사용)
- core/ 레이어를 통한 플랫폼 추상화

## 프로젝트 정보

- **경로**: `32_appuser/`
- **포트**: 8002
- **API clientType**: `app` → URL: `/app/v1/...`
- **아키텍처**: Light FSD (entities/widgets 레이어 없음)
- **UI**: 순수 Tailwind CSS (Radix UI 미사용)
- **모바일 전략**: v1(Capacitor+WebView) → v2(React Native)
- **테스트**: 미구성

## 핵심 설계 원칙: v2 전환 대비

### core/ 레이어 사용 필수

```typescript
// O: core/ 레이어를 통한 접근 (v2 재사용 가능)
import { authStorage } from '@/core/storage'
await authStorage.setToken(token)

// X: 직접 접근 금지 (v2에서 전부 수정 필요)
localStorage.setItem('token', token)
```

### v2 전환 시 재사용 가능 코드

| 재사용 가능 | 재작성 필요 |
|-------------|-------------|
| `core/` (인터페이스 동일, 구현만 교체) | `shared/ui/` (RN 컴포넌트) |
| `hooks/` (비즈니스 로직) | `routes/` (React Navigation) |
| `stores/` (상태 관리) | |
| `types/` (타입 정의) | |
| `api/client.ts` (API 호출) | |
| `@hs/common` (공유 토큰, 타입) | |

### UI 컴포넌트 규칙

```tsx
// 순수 Tailwind 사용 (Radix UI 금지)
// 터치 최적화 고려 (min-h-[44px], 충분한 터치 영역)
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'ghost'
  children: React.ReactNode
  onClick?: () => void
}

export const Button = ({ variant = 'primary', children, onClick }: ButtonProps) => {
  return (
    <button
      className={cn('min-h-[44px] px-4 rounded-lg', variantStyles[variant])}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
Button.displayName = "Button"
```

## 코딩 컨벤션

- 컴포넌트: `PascalCase.tsx`, Arrow Function, displayName 필수
- API: `camelCase.ts`
- Props 인터페이스 분리 선언
- `buildApiUrl()` 사용 → `/app/v1/...`

## Light FSD 구조

```
src/
├── app/          ← 프로바이더, 라우트 가드
├── core/         ← 플랫폼 추상화 (최우선 사용)
├── features/     ← 비즈니스 기능
│   └── {domain}/
│       ├── ui/       ← 기능 UI
│       └── model/    ← 훅, 스토어
└── shared/       ← 공통 코드
    ├── ui/       ← 순수 Tailwind 컴포넌트
    ├── api/      ← Axios 클라이언트
    ├── hooks/    ← 공통 훅
    ├── lib/      ← 유틸리티
    └── config/   ← 환경 설정
```

## 구현 절차

1. **core/ 추상화 확인**: 플랫폼 의존 로직이 있는지 확인
2. **타입 정의**: `features/{domain}/model/types.ts`
3. **API 함수**: `features/{domain}/api/` 또는 `shared/api/`
4. **모델 훅**: `features/{domain}/model/`
5. **UI 컴포넌트**: 순수 Tailwind로 구현
6. **페이지**: Next.js App Router에 페이지 추가
7. **모바일 최적화**: 터치 영역, 스크롤 동작, 뷰포트 대응

## 모바일 UX 고려사항

- 터치 타겟: 최소 44x44px
- 스크롤: 자연스러운 스크롤 동작
- 폼: 모바일 키보드 대응
- 이미지: 지연 로딩, 적절한 크기
- 네트워크: 오프라인/저속 환경 대비 로딩 상태

## 에러 처리

```typescript
import { getApiErrorMessage, parseApiError } from '@hs/common/api'
import type { ServerResponse, ApiError } from '@hs/common/types'
```

- 서버 에러 메시지 그대로 사용
- JWT 인증 오류 시 자동 로그인 리다이렉트

## 스크립트

```bash
npm run dev          # 개발 서버 (localhost:8002)
npm run build        # 개발 빌드
npm run build:prod   # 운영 빌드
npm run type-check   # 타입 체크
npm run lint         # 린트 실행
```
