# 컴포넌트 개발 가이드

> **프로젝트**: 31_webadmin (관리자 대시보드)
> **스택**: Next.js 15, Full FSD, Radix UI, shadcn/ui, DataGrid

---

## 1. UI 계층 구조

| 계층 | 위치 | 특성 | 예시 |
|------|------|------|------|
| Primitives | `shared/primitives/` | Radix UI 헤드리스 원본 | accordion, dialog, checkbox |
| UI | `shared/ui/` | Primitives + Tailwind 스타일 적용 | Button, Input, Card, Popup |
| Variants | `shared/ui/styles/` | cva 기반 스타일 변형 | button.variants.ts, input.variants.ts |
| Components | `shared/components/` | 로직 포함, 도메인 독립 | DataGrid, AutoComplete, TreeSearch, ErrorBoundary |
| Entity UI | `entities/*/ui/` | 읽기 전용, 도메인 특화 | UserCard, OrgnzCard, MobileAuthField |
| Feature UI | `features/*/ui/` | 비즈니스 로직 포함, 도메인 특화 | LoginForm, SignUpForm, FindPwForm |
| Widgets | `widgets/` | 레이아웃 단위 조합 | AdminLayout, Header, Sidebar, Footer |

---

## 2. 스타일링 패턴 (cva — class-variance-authority)

```typescript
// shared/ui/styles/button.variants.ts
import { cva } from "class-variance-authority"

export const buttonVariants = cva(
  ["inline-flex items-center justify-center", "focus-visible:outline-none focus-visible:ring-2"],
  {
    variants: {
      variant: {
        default: "bg-primary-gradient text-primary-foreground",
        secondary: "bg-primary-100 text-secondary-foreground hover:bg-primary-80",
        outline: "border border-primary bg-background text-primary",
        ghost: "bg-transparent hover:bg-muted",
      },
      size: {
        sm: "h-28 px-12 rounded-sm text-body-8",
        default: "h-40 px-16 rounded-md text-body-3",
        lg: "h-56 px-16 rounded-lg text-title-8",
      },
    },
    defaultVariants: { variant: "default", size: "default" },
  }
)
```

---

## 3. 주요 공통 컴포넌트 사용법

| 컴포넌트 | 설명 | 주요 Props |
|----------|------|-----------|
| `Button` | 기본 버튼 | variant(`default`/`secondary`/`outline`/`ghost`/`destructive`/`common`/`muted`), size(`sm`/`default`/`lg`) |
| `Input` | 입력 필드, validation 연동 | number formatting (`useNumberInput`) |
| `Popup` | 모달/다이얼로그 | Radix Dialog 기반 |
| `DataGrid` | 데이터 테이블 | TanStack Table 기반 (컬럼 고정, 정렬, 페이지네이션, 행 선택) |
| `AutoComplete` | 검색 + 다중 선택 | 드롭다운 자동완성 |
| `TreeSearch` | 트리 구조 검색 | 계층형 데이터 탐색 |
| `AlertProvider` / `ToastProvider` | 전역 알림/토스트 | Sonner 기반 |

---

## 4. 폼 컴포넌트 패턴

```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'

const schema = z.object({
  name: z.string().min(1, '이름을 입력하세요'),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
})

type FormValues = z.infer<typeof schema>

export const SomeForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm<FormValues>({
    resolver: zodResolver(schema),
  })
  // ...
}
SomeForm.displayName = "SomeForm"
```

폼 작성 시 핵심 사항:

- **유효성 검증**: Zod 스키마를 정의하고 `zodResolver`로 연동
- **타입 추론**: `z.infer<typeof schema>`로 폼 타입 자동 추출
- **displayName**: 모든 컴포넌트에 필수 지정
