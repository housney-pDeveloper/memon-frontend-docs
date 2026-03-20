# MEMON UI Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** MEMON의 전체 UI를 White 기반 + Lemon Gold 포인트 + 테이블 중심 Hybrid Minimal 디자인으로 전면 교체한다.

**Architecture:** 디자인 토큰(CSS 변수 + Tailwind preset)을 먼저 교체하고, 컴포넌트 variant를 업데이트한 뒤, 레이아웃/대시보드/상세 페이지를 순차적으로 변경한다. 마지막으로 appuser 모바일 앱을 동일한 디자인 시스템으로 업데이트한다. 테마 변경 기능(4종)은 CSS 변수 기반 data-theme 속성으로 구현한다.

**Tech Stack:** React 19, Vite, Tailwind CSS 3.4, Radix UI, CVA, Zustand, Expo/React Native

**Spec:** `01_docs/docs-claude/2026-03-21-ui-redesign-design.md`

---

## File Structure

### 신규 생성 파일

| 파일 | 책임 |
|------|------|
| `31_webadmin/src/shared/lib/theme.ts` | 테마 관리 (localStorage 저장/로드, data-theme 속성 적용) |
| `31_webadmin/src/shared/ui/stat-bar.tsx` | Stat Bar 컴포넌트 (대시보드 통계 인라인 표시) |
| `31_webadmin/src/shared/ui/custom-select.tsx` | 커스텀 셀렉트박스 (Radix UI Select 래핑) |
| `31_webadmin/src/shared/ui/period-filter.tsx` | 기간 필터 (셀렉트 + 날짜 범위 피커) |
| `31_webadmin/src/widgets/dashboard/UpcomingSection.tsx` | 다가오는 일정 테이블 섹션 |
| `31_webadmin/src/widgets/dashboard/RecentRecordsSection.tsx` | 최근 경조사 기록 테이블 섹션 |
| `31_webadmin/src/widgets/dashboard/DashboardStatBar.tsx` | 대시보드 전용 Stat Bar + 기간 필터 |
| `32_appuser/src/stores/themeStore.ts` | 모바일 테마 관리 (AsyncStorage 기반 Zustand) |
| `32_appuser/src/shared/theme.ts` | 모바일 테마 컬러 상수 정의 |

### 주요 수정 파일

| 파일 | 변경 내용 |
|------|-----------|
| `21_hs-common/src/tailwind/preset.js` | 컬러 토큰 전면 교체, 그라데이션 유틸리티 제거, 신규 타이포그래피 토큰 추가 |
| `31_webadmin/src/index.css` | CSS 변수 전면 교체, 테마 data-theme 정의, 네이티브 select 스타일 제거 |
| `31_webadmin/tailwind.config.js` | webadmin 전용 컬러 정리 |
| `31_webadmin/src/shared/ui/styles/button.variants.ts` | variant 컬러 교체 |
| `31_webadmin/src/shared/ui/styles/badge.variants.ts` | 불필요 variant 제거, 신규 variant 추가 |
| `31_webadmin/src/shared/ui/styles/checkbox.variants.ts` | 크기/컬러 교체 |
| `31_webadmin/src/shared/ui/combobox.tsx` | 기존 SelectBox 컬러 토큰 교체 |
| `31_webadmin/src/shared/ui/styles/input.variants.ts` | 컬러 토큰 교체 (CSS 변수 자동 반영 확인) |
| `31_webadmin/src/shared/ui/styles/radio.variants.ts` | 컬러 토큰 교체 (CSS 변수 자동 반영 확인) |
| `31_webadmin/src/shared/ui/pagination.tsx` | 신규 디자인 적용 |
| `31_webadmin/src/widgets/layout/AdminLayout.tsx` | 레이아웃 구조 조정 |
| `31_webadmin/src/widgets/sidebar/Sidebar.tsx` | 200px↔56px 토글, 새 컬러 |
| `31_webadmin/src/widgets/header/Header.tsx` | 미니멀 헤더로 변경 |
| `31_webadmin/src/app/(authenticated)/page.tsx` | 대시보드 구성 전면 교체 |
| `32_appuser/app/(tabs)/_layout.tsx` | 탭 바 컬러/스타일 교체 |
| `32_appuser/app/(tabs)/index.tsx` | 홈 화면 전면 재구성 |
| `32_appuser/app/(tabs)/records.tsx` | 기록 탭 리스트 재구성 |
| `32_appuser/app/(tabs)/relations.tsx` | 컬러 토큰 교체 (상세 디자인은 별도 spec) |
| `32_appuser/app/(tabs)/settings.tsx` | 컬러 토큰 교체 + 테마 선택 UI 추가 |

### 삭제 대상 파일

| 파일 | 이유 |
|------|------|
| `31_webadmin/src/widgets/dashboard/AnalyticsSection.tsx` | 차트 기반 → 테이블로 교체 |
| `31_webadmin/src/widgets/dashboard/InsightSection.tsx` | 테이블 중심 UI로 불필요 |
| `31_webadmin/src/widgets/dashboard/QuickActionsSection.tsx` | 대시보드에서 제거 |
| `31_webadmin/src/widgets/dashboard/DashboardHeader.tsx` | Welcome 영역으로 통합 |
| `31_webadmin/src/widgets/dashboard/ActivitySection.tsx` | UpcomingSection + RecentRecordsSection으로 대체 |

---

## Task 1: 디자인 토큰 교체 — hs-common Tailwind Preset

**Files:**
- Modify: `21_hs-common/src/tailwind/preset.js`

**의존:** 없음 (최초 작업)

- [ ] **Step 1: preset.js 컬러 시스템 교체**

`colors` 객체를 새 CSS 변수 기반으로 전면 교체한다:

```js
colors: {
  background: 'var(--background)',
  surface: 'var(--surface)',
  foreground: 'var(--foreground)',
  'muted-foreground': 'var(--muted-foreground)',
  placeholder: 'var(--placeholder)',
  border: 'var(--border)',
  'border-light': 'var(--border-light)',
  'row-hover': 'var(--row-hover)',
  'row-zebra': 'var(--row-zebra)',
  'row-selected': 'var(--row-selected)',
  destructive: 'var(--destructive)',
  info: 'var(--info)',
  primary: {
    DEFAULT: 'var(--primary)',
    hover: 'var(--primary-hover)',
    light: 'var(--primary-light)',
    foreground: 'var(--primary-foreground)',
  },
  ring: 'var(--ring)',
  input: 'var(--input)',
  disabled: {
    DEFAULT: 'var(--disabled)',
    foreground: 'var(--disabled-foreground)',
  },
  muted: {
    DEFAULT: 'var(--surface)',
    foreground: 'var(--muted-foreground)',
  },
  popover: {
    DEFAULT: 'var(--background)',
    foreground: 'var(--foreground)',
  },
  accent: {
    DEFAULT: 'var(--row-hover)',
    foreground: 'var(--foreground)',
  },
}
```

기존 `primary.gradient`, `primary.10/60/80/100`, `secondary`, `common`, `gray.1-4`, `point` 등 모두 제거.

- [ ] **Step 2: 그라데이션 유틸리티 플러그인 제거**

`plugins` 배열에서 `.bg-primary-gradient`, `.border-primary-gradient`, `.text-primary-gradient` 정의를 제거한다. `.scrollbar-gutter-stable` 유틸리티는 보존한다.

- [ ] **Step 3: 신규 타이포그래피 토큰 추가**

`fontSize` 객체에 다음을 추가:

```js
'caption': ['11px', { lineHeight: '16px', fontWeight: '500' }],
'sm': ['13px', { lineHeight: '20px', fontWeight: '400' }],
'stat': ['20px', { lineHeight: '28px', fontWeight: '600' }],
'page-title': ['20px', { lineHeight: '28px', fontWeight: '600' }],
'section-title': ['15px', { lineHeight: '22px', fontWeight: '600' }],
```

기존 `title-*`, `body-*`, `m-*` 토큰은 유지.

- [ ] **Step 4: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check
```

Expected: 일부 컬러 참조 에러 가능 (이후 Task에서 해결). 프리셋 자체 구문 오류 없는지 확인.

- [ ] **Step 5: 커밋**

```bash
git add 21_hs-common/src/tailwind/preset.js
git commit -m "refactor: Tailwind preset 컬러 토큰을 CSS 변수 기반으로 전면 교체"
```

---

## Task 2: CSS 변수 교체 — index.css + 테마 시스템

**Files:**
- Modify: `31_webadmin/src/index.css`
- Modify: `31_webadmin/tailwind.config.js`
- Create: `31_webadmin/src/shared/lib/theme.ts`

**의존:** Task 1

- [ ] **Step 1: index.css CSS 변수 전면 교체**

기존 `:root` 블록 내 모든 CSS 변수를 spec 9.1에 정의된 변수로 교체:

```css
:root {
  /* 공통 (불변) */
  --background: #FFFFFF;
  --surface: #FAFAFA;
  --foreground: #111827;
  --muted-foreground: #6B7280;
  --placeholder: #9CA3AF;
  --border: #E5E7EB;
  --border-light: #F3F4F6;
  --row-hover: #F3F4F6;
  --row-zebra: #F9FAFB;
  --ring: var(--primary);
  --input: #E5E7EB;
  --disabled: #F5F5F5;
  --disabled-foreground: #999999;
  --destructive: #DC2626;
  --info: #2563EB;

  /* Lemon Gold (default) */
  --primary: #EAB308;
  --primary-hover: #CA8A04;
  --primary-light: #FEF9C3;
  --primary-foreground: #111827;
  --row-selected: #FFFBEB;

  --radius: 8px;
}

[data-theme="deep-blue"] {
  --primary: #1A56DB;
  --primary-hover: #1E40AF;
  --primary-light: #DBEAFE;
  --primary-foreground: #FFFFFF;
  --row-selected: #EFF6FF;
}

[data-theme="emerald"] {
  --primary: #059669;
  --primary-hover: #047857;
  --primary-light: #D1FAE5;
  --primary-foreground: #FFFFFF;
  --row-selected: #ECFDF5;
}

[data-theme="monochrome"] {
  --primary: #111827;
  --primary-hover: #374151;
  --primary-light: #F3F4F6;
  --primary-foreground: #FFFFFF;
  --row-selected: #F9FAFB;
}
```

- [ ] **Step 2: 네이티브 select 스타일, 구 컬러 참조 제거**

`index.css`에서 다음을 제거/교체:
- 네이티브 `<select>` 스타일링 블록 제거
- 스크롤바 `var(--point-gray)` → `var(--placeholder)` 교체
- 캘린더 `bg-primary-100`, `bg-primary-10`, hardcoded `rgba(217, 200, 95, ...)` → 새 토큰으로 교체
- `.bg-primary-gradient`, `.border-primary-gradient`, `.text-primary-gradient` 사용 제거

- [ ] **Step 3: tailwind.config.js 정리**

webadmin 전용 색상(`surface`, `brand-deep`, `primary-hover`, `success`) 제거 — preset에서 모두 제공.

```js
import hsPreset from '../21_hs-common/src/tailwind/preset.js'

export default {
  presets: [hsPreset],
  content: [
    './src/app/**/*.{js,ts,jsx,tsx}',
    './src/widgets/**/*.{js,ts,jsx,tsx}',
    './src/features/**/*.{js,ts,jsx,tsx}',
    './src/entities/**/*.{js,ts,jsx,tsx}',
    './src/shared/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  plugins: [require('tailwindcss-animate')],
}
```

- [ ] **Step 4: theme.ts 생성**

```ts
// 31_webadmin/src/shared/lib/theme.ts

export type ThemeKey = 'lemon-gold' | 'deep-blue' | 'emerald' | 'monochrome';

export const THEME_STORAGE_KEY = 'memon-theme';

export const THEMES: { key: ThemeKey; label: string }[] = [
  { key: 'lemon-gold', label: 'Lemon Gold' },
  { key: 'deep-blue', label: 'Deep Blue' },
  { key: 'emerald', label: 'Emerald Green' },
  { key: 'monochrome', label: 'Monochrome' },
];

export function getStoredTheme(): ThemeKey {
  const stored = localStorage.getItem(THEME_STORAGE_KEY);
  if (stored && THEMES.some((t) => t.key === stored)) {
    return stored as ThemeKey;
  }
  return 'lemon-gold';
}

export function applyTheme(theme: ThemeKey): void {
  const root = document.documentElement;
  if (theme === 'lemon-gold') {
    root.removeAttribute('data-theme');
  } else {
    root.setAttribute('data-theme', theme);
  }
  localStorage.setItem(THEME_STORAGE_KEY, theme);
}

export function initTheme(): void {
  applyTheme(getStoredTheme());
}
```

- [ ] **Step 5: 앱 진입점에서 initTheme 호출**

`src/main.tsx` 또는 root provider에서 `initTheme()` 호출 추가.

- [ ] **Step 6: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 7: 커밋**

```bash
git add 31_webadmin/src/index.css 31_webadmin/tailwind.config.js 31_webadmin/src/shared/lib/theme.ts
git commit -m "refactor: CSS 변수 전면 교체 및 4종 테마 시스템 구현"
```

---

## Task 3: UI 컴포넌트 Variant 업데이트

**Files:**
- Modify: `31_webadmin/src/shared/ui/styles/button.variants.ts`
- Modify: `31_webadmin/src/shared/ui/styles/badge.variants.ts`
- Modify: `31_webadmin/src/shared/ui/styles/checkbox.variants.ts`
- Modify: `31_webadmin/src/shared/ui/combobox.tsx` (기존 SelectBox)
- Modify: `31_webadmin/src/shared/ui/pagination.tsx`
- Verify: `31_webadmin/src/shared/ui/styles/input.variants.ts` (CSS 변수 자동 반영 확인)
- Verify: `31_webadmin/src/shared/ui/styles/radio.variants.ts` (CSS 변수 자동 반영 확인)

**의존:** Task 2

- [ ] **Step 1: button.variants.ts 교체**

```ts
export const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition-colors duration-150 ease-in-out focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary-hover',
        secondary: 'border border-border bg-background text-foreground hover:bg-row-hover',
        destructive: 'bg-destructive text-white hover:bg-red-700',
        ghost: 'bg-transparent hover:bg-row-hover text-foreground',
      },
      size: {
        sm: 'h-32 px-12 text-sm rounded-md',
        default: 'h-40 px-16 text-sm rounded-md',
        lg: 'h-48 px-20 text-section-title rounded-lg',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  }
);
```

기존 `outline`, `common`, `muted` variant 제거.

- [ ] **Step 2: badge.variants.ts 간소화**

29+ variant를 핵심 variant만으로 축소:

```ts
export const badgeVariants = cva(
  'inline-flex items-center rounded px-8 py-2 text-caption font-medium',
  {
    variants: {
      variant: {
        default: 'bg-border-light text-foreground',
        primary: 'bg-primary-light text-foreground',
        destructive: 'bg-[#FEF2F2] text-destructive',
        info: 'bg-[#EFF6FF] text-info',
        outline: 'border border-border bg-background text-foreground',
      },
    },
    defaultVariants: { variant: 'default' },
  }
);
```

- [ ] **Step 3: checkbox.variants.ts 교체**

```ts
export const checkboxVariants = cva(
  'h-14 w-14 shrink-0 rounded-[3px] border-[1.5px] transition-colors duration-150',
  {
    variants: {
      state: {
        unchecked: 'border-[#D1D5DB] bg-background',
        checked: 'border-primary bg-primary text-white',
        disabled: 'border-input bg-disabled cursor-not-allowed',
      },
    },
    defaultVariants: { state: 'unchecked' },
  }
);
```

- [ ] **Step 4: combobox.tsx (SelectBox) 스타일 교체**

기존 combobox.tsx의 스타일을 새 컬러 토큰에 맞게 교체:
- Trigger: `border border-border rounded-md px-12 py-7 text-sm bg-background`
- Content: `border border-border rounded-lg shadow-[0_4px_16px_rgba(0,0,0,0.08)]`
- Item: `hover:bg-row-hover`
- Item selected: `bg-foreground text-white`

- [ ] **Step 5: pagination.tsx 교체**

현재 페이지: `bg-foreground text-white w-32 h-32 rounded-md`
비활성: `text-muted-foreground w-32 h-32 rounded-md`
화살표: chevron 아이콘, `text-placeholder`

- [ ] **Step 6: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 7: 커밋**

```bash
git add 31_webadmin/src/shared/ui/styles/ 31_webadmin/src/shared/ui/select.tsx 31_webadmin/src/shared/ui/pagination.tsx
git commit -m "refactor: UI 컴포넌트 variant를 White 기반 디자인으로 교체"
```

---

## Task 4: 레이아웃 변경 — Sidebar + Header + AdminLayout

**Files:**
- Modify: `31_webadmin/src/widgets/sidebar/Sidebar.tsx`
- Modify: `31_webadmin/src/widgets/header/Header.tsx`
- Modify: `31_webadmin/src/widgets/layout/AdminLayout.tsx`
- Modify: `31_webadmin/src/shared/config/menu-config.ts`

**의존:** Task 2, Task 3

- [ ] **Step 1: Sidebar.tsx 재구현**

핵심 변경:
- 너비: 240px → 200px (확장) / 72px → 56px (축소)
- 배경: `bg-surface` (#FAFAFA)
- 로고: 28×28px `bg-primary` 배경 + 흰색 "M" + "MEMON" 텍스트
- 활성 메뉴: `bg-foreground text-white rounded-md` (테마 무관 고정)
- 비활성: `text-muted-foreground hover:bg-row-hover`
- 하단 토글: 접기/펼치기 버튼
- 반응형: 1024px 이하 자동 축소, 768px 이하 오버레이
- 트랜지션: `transition-all duration-200 ease-in-out`

- [ ] **Step 2: Header.tsx 미니멀 재구현**

핵심 변경:
- 높이: 52px
- 좌측: 브레드크럼만 (AutoBreadcrumb)
- 우측: 알림 아이콘(뱃지) + 프로필 아바타
- 기존 사용자명 텍스트, logout 버튼, divider 제거
- 배경: `bg-background`, 하단 보더: `border-b border-border`

- [ ] **Step 3: AdminLayout.tsx 조정**

사이드바 너비 변경에 맞춰 content area 패딩 조정:
- Content padding: `p-24`
- Sidebar 상태에 따른 margin-left 자동 조정

- [ ] **Step 4: menu-config.ts 아이콘 정리**

메뉴 아이템 아이콘을 Lucide 아이콘으로 통일 (현재도 Lucide 사용 중이므로 확인만):
- Dashboard: LayoutDashboard
- Records: FileText
- Relations: Users
- Ceremonies 제거 (Records에 통합) 또는 유지 — 기존 라우트 구조 유지
- Schedules: Calendar
- Statistics: BarChart3
- Settings: Settings

- [ ] **Step 5: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 6: 커밋**

```bash
git add 31_webadmin/src/widgets/ 31_webadmin/src/shared/config/menu-config.ts
git commit -m "refactor: Sidebar(200/56px 토글) + Header 미니멀 재구현"
```

---

## Task 5: 신규 공통 컴포넌트 — StatBar, CustomSelect, PeriodFilter

**Files:**
- Create: `31_webadmin/src/shared/ui/stat-bar.tsx`
- Create: `31_webadmin/src/shared/ui/custom-select.tsx`
- Create: `31_webadmin/src/shared/ui/period-filter.tsx`

**의존:** Task 2, Task 3

- [ ] **Step 1: stat-bar.tsx 생성**

Stat Bar 컴포넌트:
- Props: `items: { label: string; value: string; sub?: string; valueClassName?: string }[]`
- 레이아웃: flex, gap-[1px], bg-border, rounded-lg, overflow-hidden, border border-border
- 각 셀: flex-1 bg-background p-[16px_20px]
- label: text-caption text-placeholder
- value: text-stat text-foreground
- sub: text-caption (색상 prop으로 지정)

- [ ] **Step 2: custom-select.tsx 생성**

Radix UI Select 래핑 컴포넌트:
- Trigger: border-border, rounded-md, px-12 py-7, text-sm, chevron-down 아이콘
- Content: border-border, rounded-lg, shadow, bg-background
- Item hover: `bg-row-zebra` (#F9FAFB, spec 5.3과 일치)
- Item selected: `bg-foreground text-white`
- 드롭다운 열기: `transition 150ms ease-out`, 닫기: `100ms ease-in`
- 네이티브 `<select>` 절대 사용 금지

- [ ] **Step 3: period-filter.tsx 생성**

기간 필터 컴포넌트:
- CustomSelect: 전체/이번달/지난달/올해/작년
- DateRangePicker: 캘린더 아이콘 + 시작일 — 종료일
- 셀렉트 변경 시 날짜 범위 자동 연동
- Props: `onDateRangeChange: (start: Date, end: Date) => void`

- [ ] **Step 4: shared/ui/index.ts barrel export 업데이트**

`31_webadmin/src/shared/ui/index.ts`에 신규 컴포넌트 export 추가:
```ts
export { StatBar } from './stat-bar';
export { CustomSelect } from './custom-select';
export { PeriodFilter } from './period-filter';
```

- [ ] **Step 5: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 6: 커밋**

```bash
git add 31_webadmin/src/shared/ui/stat-bar.tsx 31_webadmin/src/shared/ui/custom-select.tsx 31_webadmin/src/shared/ui/period-filter.tsx 31_webadmin/src/shared/ui/index.ts
git commit -m "feat: StatBar, CustomSelect, PeriodFilter 공통 컴포넌트 추가"
```

---

## Task 6: 대시보드 전면 재구성

**Files:**
- Create: `31_webadmin/src/widgets/dashboard/DashboardStatBar.tsx`
- Create: `31_webadmin/src/widgets/dashboard/UpcomingSection.tsx`
- Create: `31_webadmin/src/widgets/dashboard/RecentRecordsSection.tsx`
- Modify: `31_webadmin/src/widgets/dashboard/WelcomeSection.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/page.tsx`
- Delete: `31_webadmin/src/widgets/dashboard/AnalyticsSection.tsx`
- Delete: `31_webadmin/src/widgets/dashboard/InsightSection.tsx`
- Delete: `31_webadmin/src/widgets/dashboard/QuickActionsSection.tsx`
- Delete: `31_webadmin/src/widgets/dashboard/DashboardHeader.tsx`

**의존:** Task 4, Task 5

- [ ] **Step 1: WelcomeSection.tsx 간소화**

KPI 카드 5개 제거. 인사말 + 날짜만 유지:
- 인사말: text-page-title font-semibold text-foreground
- 날짜: text-sm text-placeholder, yyyy년 m월 d일 요일 형식

- [ ] **Step 2: DashboardStatBar.tsx 생성**

PeriodFilter + StatBar 조합:
- PeriodFilter: 상단 (셀렉트 + 날짜 범위)
- StatBar: 4분할 (총 보낸 금액 / 총 받은 금액 / 차액 / 이번 달 예정)
- 차액 음수: `text-destructive`
- 이번 달 예정 금액: `text-primary`
- API: `useHomeSummary()` 훅 활용

- [ ] **Step 3: UpcomingSection.tsx 생성**

다가오는 일정 테이블:
- 헤더: "다가오는 일정" + "전체 보기 →" 링크
- 테이블 컬럼: 날짜(140px) | D-day(56px) | 행사(flex) | 대상(100px) | 관계(80px badge) | 예상 금액(120px 우측정렬)
- 날짜 형식: yyyy.mm.dd (요일)
- D-day 3일 이내: `text-primary font-semibold`, 나머지: `text-placeholder`
- 교대 행: `even:bg-row-zebra`
- 테이블 스타일: `border border-border rounded-lg overflow-hidden`
- 헤더 행: `bg-surface text-caption text-placeholder font-medium`

- [ ] **Step 4: RecentRecordsSection.tsx 생성**

최근 경조사 기록 테이블:
- 헤더: "최근 경조사 기록" + "전체 보기 →" 링크
- 테이블 컬럼: 날짜(140px) | 행사(flex) | 대상(100px) | 관계(80px badge) | 방향(80px) | 금액(120px 우측정렬)
- 방향 "보냄": `text-destructive`, "받음": `text-info`
- 금액 색상: 방향에 따라 destructive/info
- 동일 테이블 스타일 적용

- [ ] **Step 5: page.tsx 재구성**

```tsx
<div className="space-y-24">
  <WelcomeSection />
  <DashboardStatBar />
  <UpcomingSection />
  <RecentRecordsSection />
</div>
```

- [ ] **Step 6: 구 대시보드 위젯 삭제**

AnalyticsSection.tsx, ActivitySection.tsx, InsightSection.tsx, QuickActionsSection.tsx, DashboardHeader.tsx 삭제.
이 파일들을 import하는 곳이 있으면 함께 제거.

- [ ] **Step 7: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint && npm run build
```

- [ ] **Step 8: 커밋**

```bash
git add 31_webadmin/src/widgets/dashboard/ 31_webadmin/src/app/(authenticated)/page.tsx
git commit -m "feat: 대시보드를 StatBar + 테이블 중심 Hybrid Minimal로 전면 재구성"
```

---

## Task 7: 경조사 기록 상세 페이지 테이블

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/records/` (해당 페이지 파일)

**의존:** Task 5, Task 6

- [ ] **Step 1: 기록 목록 페이지 재구현**

페이지 구성:
1. 헤더: "경조사 기록" (text-page-title) + "새 기록 추가" CTA 버튼 (bg-primary)
2. 필터바: 검색(debounce 300ms) + 행사유형/관계/방향 CustomSelect + 날짜범위 + 총 건수
3. 테이블: 체크박스(36px) | 날짜(140px) | 행사(flex) | 대상(110px) | 관계(80px) | 방향(80px) | 금액(120px) | 액션(48px)
4. 행 상태: Default(bg-background) / Zebra(bg-row-zebra) / Hover(bg-row-hover) / Selected(bg-row-selected)
5. 정렬: 날짜, 금액 컬럼 ASC/DESC 토글
6. 행 클릭: 상세 보기 페이지로 이동 (체크박스 영역 제외)
7. 행 액션 메뉴 (⋮): 수정/상세보기/삭제
8. 페이지네이션 컴포넌트 사용

- [ ] **Step 2: 다중 선택 기능**

체크박스 선택 시:
- 선택된 행: `bg-row-selected`
- 하단: "N건 선택됨" + 일괄 삭제 버튼

- [ ] **Step 3: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 4: 커밋**

```bash
git add 31_webadmin/src/app/(authenticated)/records/
git commit -m "feat: 경조사 기록 페이지를 필터+테이블+페이지네이션 구조로 재구현"
```

---

## Task 8: 설정 페이지 — 테마 선택 UI

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/settings/` (해당 페이지 파일)

**의존:** Task 2, Task 3

- [ ] **Step 1: 테마 선택 섹션 추가**

설정 페이지에 "테마" 섹션 추가:
- 4개 테마 프리뷰 (Lemon Gold / Deep Blue / Emerald Green / Monochrome)
- 각 프리뷰: 컬러 swatch + 테마명 + 미니 버튼 프리뷰
- 현재 선택된 테마: border-primary 강조
- 클릭 시 `applyTheme()` 호출 → 즉시 적용

- [ ] **Step 2: 빌드 검증**

```bash
cd 31_webadmin && npm run type-check && npm run lint
```

- [ ] **Step 3: 커밋**

```bash
git add 31_webadmin/src/app/(authenticated)/settings/
git commit -m "feat: 설정 페이지에 4종 테마 선택 UI 추가"
```

---

## Task 9: 모바일 appuser — 테마 시스템 + 컬러 교체

**Files:**
- Create: `32_appuser/src/shared/theme.ts`
- Create: `32_appuser/src/stores/themeStore.ts`
- Modify: `32_appuser/app/(tabs)/_layout.tsx`

**의존:** 없음 (독립 작업, Task 1-8과 병렬 가능)

- [ ] **Step 1: theme.ts 컬러 상수 정의**

```ts
// 32_appuser/src/shared/theme.ts

export type ThemeKey = 'lemon-gold' | 'deep-blue' | 'emerald' | 'monochrome';

export const THEMES = {
  'lemon-gold': {
    primary: '#EAB308',
    primaryHover: '#CA8A04',
    primaryLight: '#FEF9C3',
    primaryForeground: '#111827',
    rowSelected: '#FFFBEB',
  },
  'deep-blue': {
    primary: '#1A56DB',
    primaryHover: '#1E40AF',
    primaryLight: '#DBEAFE',
    primaryForeground: '#FFFFFF',
    rowSelected: '#EFF6FF',
  },
  emerald: {
    primary: '#059669',
    primaryHover: '#047857',
    primaryLight: '#D1FAE5',
    primaryForeground: '#FFFFFF',
    rowSelected: '#ECFDF5',
  },
  monochrome: {
    primary: '#111827',
    primaryHover: '#374151',
    primaryLight: '#F3F4F6',
    primaryForeground: '#FFFFFF',
    rowSelected: '#F9FAFB',
  },
} as const;

export const COMMON_COLORS = {
  background: '#FFFFFF',
  surface: '#FAFAFA',
  foreground: '#111827',
  mutedForeground: '#6B7280',
  placeholder: '#9CA3AF',
  border: '#E5E7EB',
  borderLight: '#F3F4F6',
  rowHover: '#F3F4F6',
  rowZebra: '#F9FAFB',
  destructive: '#DC2626',
  info: '#2563EB',
} as const;
```

- [ ] **Step 2: themeStore.ts 생성**

```ts
// 32_appuser/src/stores/themeStore.ts

import { create } from 'zustand';
import { createJSONStorage, persist } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { type ThemeKey, THEMES } from '@/shared/theme';

interface ThemeState {
  theme: ThemeKey;
  setTheme: (theme: ThemeKey) => void;
  getColors: () => (typeof THEMES)[ThemeKey];
}

export const useThemeStore = create<ThemeState>()(
  persist(
    (set, get) => ({
      theme: 'lemon-gold',
      setTheme: (theme) => set({ theme }),
      getColors: () => THEMES[get().theme],
    }),
    {
      name: 'memon-theme',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

- [ ] **Step 3: _layout.tsx 탭 바 컬러 교체**

기존 hardcoded 컬러를 themeStore 기반으로 교체:
- `tabBarActiveTintColor`: `theme.primary` 대신 `#111827` (고정)
- `tabBarInactiveTintColor`: `#9CA3AF`
- `tabBarStyle`: height 56px, borderColor `#E5E7EB`
- 아이콘: Ionicons 유지

- [ ] **Step 4: 빌드 검증**

```bash
cd 32_appuser && npx tsc --noEmit
```

- [ ] **Step 5: 커밋**

```bash
git add 32_appuser/src/shared/theme.ts 32_appuser/src/stores/themeStore.ts 32_appuser/app/(tabs)/_layout.tsx
git commit -m "feat: appuser 테마 시스템 구현 및 탭 바 컬러 교체"
```

---

## Task 10: 모바일 appuser — 홈 화면 재구성

**Files:**
- Modify: `32_appuser/app/(tabs)/index.tsx`

**의존:** Task 9

- [ ] **Step 1: 홈 화면 전면 재구현**

구성 (위에서 아래):
1. Header: 로고(M+MEMON) + 알림 아이콘
2. Welcome: 인사말 + 날짜 `yyyy.mm.dd (요일)`
3. Stat Bar: 3분할 (보낸 금액 / 받은 금액 / 차액)
   - 만원 단위 축약: `325만원`, `187만원`, `138만원`
   - 1만원 미만은 원 단위 그대로
   - 차액 음수: `color: COMMON_COLORS.destructive`
4. 다가오는 일정 리스트: 행사·대상 1줄 + 날짜·관계 2줄, D-day 우측
5. 최근 기록 리스트: 동일 패턴, 금액 우측

스타일:
- 배경: `COMMON_COLORS.background`
- 리스트 아이템: border borderLight, 교대 배경 rowZebra
- 금액 보냄: destructive, 받음: info

- [ ] **Step 2: 금액 축약 유틸 함수**

```ts
function formatManwon(amount: number): string {
  const abs = Math.abs(amount);
  if (abs >= 10000) {
    return `${Math.floor(abs / 10000)}만원`;
  }
  return `${abs.toLocaleString()}원`;
}
```

- [ ] **Step 3: 빌드 검증**

```bash
cd 32_appuser && npx tsc --noEmit
```

- [ ] **Step 4: 커밋**

```bash
git add 32_appuser/app/(tabs)/index.tsx
git commit -m "feat: appuser 홈 화면을 StatBar + 리스트 중심으로 재구성"
```

---

## Task 11: 모바일 appuser — 기록 탭 재구성

**Files:**
- Modify: `32_appuser/app/(tabs)/records.tsx`

**의존:** Task 9

- [ ] **Step 1: 기록 탭 전면 재구현**

구성:
1. Header: "경조사 기록" + FAB(원형, primary 배경, + 아이콘)
2. 검색 + 필터 아이콘(bottom sheet)
3. 기간 pill 탭: 전체/이번달/올해/작년 (활성: bg-foreground text-white, 비활성: bg-borderLight text-mutedForeground)
4. 월별 sticky 그룹 헤더: "2026년 3월" (bg-surface, text-caption)
5. 리스트 아이템: 행사·대상 1줄 + 날짜(요일)·관계 badge 2줄 | 금액+방향 우측
6. 좌 스와이프: 수정(파란)/삭제(빨간) 액션

- [ ] **Step 2: 빌드 검증**

```bash
cd 32_appuser && npx tsc --noEmit
```

- [ ] **Step 3: 커밋**

```bash
git add 32_appuser/app/(tabs)/records.tsx
git commit -m "feat: appuser 기록 탭을 월별 그룹 리스트 + pill 필터로 재구성"
```

---

## Task 12: 모바일 appuser — 관계/설정 탭 컬러 교체

**Files:**
- Modify: `32_appuser/app/(tabs)/relations.tsx`
- Modify: `32_appuser/app/(tabs)/settings.tsx`

**의존:** Task 9

- [ ] **Step 1: relations.tsx 컬러 토큰 교체**

기존 hardcoded 컬러(#D9C85F, #4C5332 등)를 `COMMON_COLORS` / `themeStore.getColors()`로 교체.
상세 UI 디자인은 별도 spec에서 정의 — 이 Task에서는 컬러 토큰만 교체.

- [ ] **Step 2: settings.tsx 테마 선택 UI 추가**

설정 화면에 "테마" 섹션 추가:
- 4개 테마 프리뷰 (컬러 swatch + 테마명)
- 현재 선택된 테마 체크 표시
- 탭 시 `themeStore.setTheme()` 호출

- [ ] **Step 3: 빌드 검증**

```bash
cd 32_appuser && npx tsc --noEmit
```

- [ ] **Step 4: 커밋**

```bash
git add 32_appuser/app/(tabs)/relations.tsx 32_appuser/app/(tabs)/settings.tsx
git commit -m "feat: appuser 관계/설정 탭 컬러 교체 및 테마 선택 UI 추가"
```

---

## Task 13: 전체 빌드 검증 & 정리

**Files:** 전체

**의존:** Task 1-12 전체 완료

- [ ] **Step 1: webadmin 전체 빌드**

```bash
cd 31_webadmin && npm run type-check && npm run lint && npm run build
```

- [ ] **Step 2: appuser 타입 체크**

```bash
cd 32_appuser && npx tsc --noEmit
```

- [ ] **Step 3: 미사용 import/변수 정리**

빌드 에러나 lint 경고로 발견된 미사용 참조 정리.

- [ ] **Step 4: 최종 커밋**

```bash
git add .
git commit -m "chore: UI 리디자인 전체 빌드 검증 및 미사용 코드 정리"
```

---

## 실행 순서 요약

```
Task 1 (Preset) ─── Task 2 (CSS + Theme) ─── Task 3 (Variants) ─┐
                                                                   ├── Task 4 (Layout) ─┐
                                                                   ├── Task 5 (Components) ─┤
                                                                   │                         ├── Task 6 (Dashboard)
                                                                   │                         ├── Task 7 (Records)
                                                                   └── Task 8 (Settings)     │

Task 9 (Mobile Theme) ─┬── Task 10 (Mobile Home)
                        ├── Task 11 (Mobile Records)
                        └── Task 12 (Mobile Relations/Settings)

Task 13 (Final Build) ── depends on all
```

**병렬 가능:** Task 9-12 (모바일)은 Task 1-8 (웹)과 독립적으로 병렬 수행 가능.
