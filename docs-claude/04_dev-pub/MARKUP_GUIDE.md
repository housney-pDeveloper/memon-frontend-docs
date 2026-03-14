# 마크업 및 스타일링 가이드

> 11_publishing 프로젝트의 HTML 마크업 및 Tailwind CSS 스타일링 규칙

---

## 1. Tailwind CSS 사용 규칙

- **인라인 스타일 금지** — Tailwind 유틸리티 클래스만 사용
- **`cn()` 함수**로 조건부 클래스 결합 (`clsx` + `twMerge`)
- **hs-common Tailwind 프리셋** 적용 (`tailwind.config.js`)
- 커스텀 CSS 최소화 — Tailwind 유틸리티 우선

---

## 2. 디자인 토큰 활용

```javascript
// tailwind.config.js
import hsPreset from './hs-common/src/tailwind/preset.js'

export default {
  presets: [hsPreset],
  // 프리셋에서 제공하는 값 사용:
  // - colors: primary, secondary, destructive, gray 등
  // - typography: text-title-*, text-body-* 등
  // - spacing: Tailwind 기본 + 커스텀
}
```

- 색상, 타이포그래피, 간격 등은 프리셋에서 제공하는 디자인 토큰을 활용
- 직접 색상 값(`#FF0000`, `rgb(...)`)을 하드코딩하지 않음

---

## 3. 반응형 디자인

```html
<!-- 모바일 퍼스트 접근 -->
<div class="px-4 md:px-6 lg:px-8">
  <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
    ...
  </div>
</div>
```

### 브레이크포인트

| 접두사 | 최소 너비 |
|--------|-----------|
| `sm` | 640px |
| `md` | 768px |
| `lg` | 1024px |
| `xl` | 1280px |
| `2xl` | 1536px |

### 원칙

- **모바일 퍼스트** — 기본 스타일은 모바일 기준, 브레이크포인트로 확장
- 클래스 없는 상태가 모바일 레이아웃이 되어야 함

---

## 4. 컴포넌트 스타일 패턴 (cva)

```typescript
import { cva, type VariantProps } from "class-variance-authority"

const cardVariants = cva("rounded-lg border", {
  variants: {
    variant: {
      default: "bg-white border-gray-200",
      outlined: "bg-transparent border-gray-300",
      elevated: "bg-white border-transparent shadow-md",
    },
    padding: {
      sm: "p-3",
      md: "p-4",
      lg: "p-6",
    },
  },
  defaultVariants: { variant: "default", padding: "md" },
})
```

- `class-variance-authority`를 사용하여 변형(variant) 기반 스타일 관리
- `defaultVariants`로 기본값 지정

---

## 5. 시맨틱 HTML 구조

```html
<header>
  <nav>...</nav>
</header>
<main>
  <section>
    <article>...</article>
  </section>
  <aside>...</aside>
</main>
<footer>...</footer>
```

- 페이지 구조를 의미에 맞는 태그로 구성
- `<div>` 남용 금지 — 적절한 시맨틱 태그 우선 사용
- 접근성을 고려한 마크업 작성

---

## 6. 아이콘

- **Lucide React** 사용 (`lucide-react`)
- 일관된 크기 사용: `size={16}`, `size={20}`, `size={24}`
- 접근성 속성 제공:
  - 장식용 아이콘: `aria-hidden="true"`
  - 의미 있는 아이콘: `aria-label` 제공

```tsx
import { Search, ChevronRight } from "lucide-react"

// 장식용
<Search size={20} aria-hidden="true" />

// 의미 있는 아이콘
<ChevronRight size={16} aria-label="다음으로 이동" />
```

---

## 7. 이미지

- **Next.js `Image` 컴포넌트** 사용
- `alt` 속성 필수
- 적절한 `width`/`height` 지정
- 지연 로딩 활용 (`loading="lazy"`)

```tsx
import Image from "next/image"

<Image
  src="/images/banner.png"
  alt="메인 배너 이미지"
  width={1200}
  height={400}
  loading="lazy"
/>
```

---

## 8. webadmin / appuser 분리 퍼블리싱

| 구분 | 컴포넌트 사용 규칙 |
|------|---------------------|
| **webadmin용 퍼블리싱** | Radix UI 컴포넌트 사용 가능 |
| **appuser용 퍼블리싱** | 순수 Tailwind CSS만 사용 (Radix UI 금지) |

### 규칙

- 어느 플랫폼용인지 **페이지/섹션에 명시**
- webadmin은 shadcn/ui 기반 컴포넌트를 적극 활용
- appuser는 Radix UI 의존 없이 순수 HTML + Tailwind CSS로 마크업
- 두 플랫폼 간 컴포넌트 혼용 금지
