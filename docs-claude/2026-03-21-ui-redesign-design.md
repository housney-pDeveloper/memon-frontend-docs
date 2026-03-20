# MEMON UI Redesign — Design Spec

**Date:** 2026-03-21
**Scope:** 31_webadmin (Vite+React) + 32_appuser (Expo+React Native)
**Approach:** Hybrid Minimal — Stat Bar + 테이블 중심

---

## 1. 디자인 원칙

1. **White base** — 순수 흰색(#FFFFFF) 배경, 컬러 사용 최소화
2. **정갈한 신뢰감** — Pretendard 폰트, 정돈된 테이블 레이아웃, 깔끔한 여백
3. **White/Black 우선** — 액센트 컬러는 CTA 버튼, 활성 상태, 금전 방향 등 핵심 포인트에만 사용
4. **테이블 중심** — 카드 UI 지양, 데이터를 한 눈에 볼 수 있는 테이블/리스트 구조
5. **현대적 UX** — 마이크로 인터랙션, 적절한 여백, hover/transition으로 칙칙하지 않은 경험
6. **한 화면 액센트 3곳 이하** — 초과 시 과다 컬러 사용으로 간주, 검토 필요

---

## 2. 컬러 시스템

### 2.1 공통 컬러 (테마 불변)

| 토큰 | 값 | 용도 |
|------|------|------|
| `--background` | #FFFFFF | 페이지 배경 |
| `--surface` | #FAFAFA | 사이드바, 테이블 헤더 |
| `--foreground` | #111827 | 본문 텍스트 |
| `--muted-foreground` | #6B7280 | 보조 텍스트, 라벨 |
| `--placeholder` | #9CA3AF | placeholder, 비활성 텍스트 |
| `--border` | #E5E7EB | 테이블 외곽, 구분선 |
| `--border-light` | #F3F4F6 | 테이블 행 구분, 리스트 구분선 |
| `--row-hover` | #F3F4F6 | 테이블 행 hover |
| `--row-zebra` | #F9FAFB | 짝수 행 배경 |
| `--ring` | var(--primary) | 포커스 링 |
| `--input` | #E5E7EB | input 보더 (--border와 동일) |
| `--disabled` | #F5F5F5 | 비활성 배경 |
| `--disabled-foreground` | #999999 | 비활성 텍스트 |
| `--destructive` | #DC2626 | 에러, 삭제, 보낸 금액 |
| `--info` | #2563EB | 받은 금액 |

### 2.2 시맨틱 컬러 — 금전 이동 방향 (테마 불변)

| 방향 | 텍스트 컬러 | 배경 (뱃지용) |
|------|------------|-------------|
| 보냄 (지출) | #DC2626 | #FEF2F2 |
| 받음 (수입) | #2563EB | #EFF6FF |

### 2.3 테마별 액센트 컬러 (사용자 선택 가능)

**기본 테마: Lemon Gold**

| 테마 | Primary | Hover | Light | 비고 |
|------|---------|-------|-------|------|
| Lemon Gold (default) | #EAB308 | #CA8A04 | #FEF9C3 | MEMON↔LEMON 브랜드 연결 |
| Deep Blue | #1A56DB | #1E40AF | #DBEAFE | 핀테크 신뢰 컬러 |
| Emerald Green | #059669 | #047857 | #D1FAE5 | 금전·성장 상징 |
| Monochrome | #111827 | #374151 | #F3F4F6 | 순수 Black/Gray |

**테마 적용 범위:**
- CTA 버튼 배경: `--primary`
- CTA 버튼 hover: `--primary-hover`
- 체크박스 선택 상태: `--primary`
- 행 선택 배경: `--row-selected` (Lemon Gold 시 #FFFBEB, `--primary-light`와 별도 토큰)
- 활성 네비게이션 아이콘/텍스트: 사이드바는 #111827 고정 (테마 무관)
- 로고: `--primary`
- D-day 임박 강조: `--primary`

**테마 변경 위치:** 설정 페이지

### 2.4 컬러 사용 규칙

- **허용:** CTA 버튼, 활성 네비게이션, 로고, 선택 상태, 금전 방향, D-day 임박
- **금지:** 배경색 채우기, 장식용 컬러, 그라데이션, 다색 차트
- 기존 Lemon Palette(#D9C85F) 기반 gradient, brand-deep, success 등 모두 제거

---

## 3. 타이포그래피

### 3.1 폰트

- **Primary:** Pretendard (Regular 400, Medium 500, SemiBold 600)
- **로딩:** @font-face, WOFF2 format
- **Fallback:** -apple-system, BlinkMacSystemFont, system-ui, Roboto, sans-serif
- **호환:** 모든 브라우저/OS 지원

### 3.2 타이포그래피 토큰 매핑

기존 프리셋 타이포그래피에 없는 크기를 추가한다:

| UI 용도 | 크기 | Weight | Tailwind 토큰 |
|---------|------|--------|--------------|
| Stat Bar 라벨, 테이블 헤더, badge | 11px / 16px LH | 500 | `text-caption` (신규) |
| 테이블 본문, 필터 텍스트, 날짜 | 12-13px / 20px LH | 400-500 | `text-body-8` (기존 15px→13px 변경) 또는 `text-sm` (신규) |
| Stat Bar 금액 | 20px / 28px LH | 600 | `text-stat` (신규) |
| 페이지 제목 | 20px / 28px LH | 600 | `text-page-title` (신규) |
| 섹션 제목 | 15px / 22px LH | 600 | `text-section-title` (신규) |

기존 `text-title-*`, `text-body-*` 토큰은 유지하되, 위 신규 토큰을 preset.js에 추가한다.

### 3.3 날짜 표기 형식

- **표준:** `yyyy.mm.dd (요일)` — 예: `2026.03.15 (토)`
- 모든 화면에서 통일 적용 (webadmin + appuser)

### 3.4 금액 표기

- **webadmin:** 원 단위 전체 표시 — `₩3,250,000`, `-₩50,000`, `+₩100,000`
- **appuser Stat Bar:** 만원 단위 축약 — `325만원`, `187만원`, `138만원`
  - 1만원 이상: "만원" 단위 축약
  - 1만원 미만: 원 단위 그대로 표시
- **appuser 리스트:** 원 단위 전체 표시 — `-₩50,000`, `+₩100,000`

---

## 4. 레이아웃

### 4.1 webadmin 레이아웃 구조

```
┌─────────────────────────────────────────────────┐
│ Sidebar (200px ↔ 56px)  │  Header (52px)        │
│                          ├───────────────────────│
│ Logo + Brand             │  Content Area         │
│ Navigation Menu          │  (padding: 24px)      │
│ ─────────────            │                       │
│ Collapse Toggle          │                       │
└─────────────────────────────────────────────────┘
```

**Sidebar (Collapsible):**
- 확장: 200px (아이콘 + 텍스트)
- 축소: 56px (아이콘만)
- 배경: #FAFAFA
- 보더: 우측 1px #E5E7EB
- 로고: 28×28px Lemon Gold 배경 + 흰색 "M"
- 활성 메뉴: #111827 배경, 흰색 텍스트 (테마 무관)
- 비활성 메뉴: #6B7280 텍스트
- 하단: 접기/펼치기 토글
- **반응형:** 1024px 이하에서 자동 축소(56px), 768px 이하에서 오버레이 + 백드롭

**Header:**
- 높이: 52px
- 좌측: 브레드크럼 (경로 표시)
- 우측: 알림 아이콘(뱃지) + 프로필 아바타
- 하단 보더: 1px #E5E7EB

**Navigation 메뉴:**
- 대시보드
- 경조사 기록
- 관계 관리
- 일정
- 통계
- 설정

### 4.2 appuser 레이아웃 구조

```
┌──────────────────┐
│ Status Bar       │
│ Header (Logo)    │
├──────────────────┤
│                  │
│ Content Area     │
│ (scrollable)     │
│                  │
├──────────────────┤
│ Tab Bar (56px)   │
└──────────────────┘
```

**Tab Bar:**
- 4탭: 홈 / 관계 / 기록 / 설정
- 높이: 56px
- 활성: #111827 (아이콘 + 텍스트)
- 비활성: #9CA3AF

---

## 5. 대시보드 (webadmin)

### 5.1 구성 (위에서 아래 순서)

1. **Welcome 영역** — 인사말(20px SemiBold) + 오늘 날짜(13px #9CA3AF)
2. **기간 필터** — 커스텀 셀렉트(전체/이번달/지난달/올해/작년) + 날짜 범위 캘린더 피커
3. **Stat Bar** — 총 보낸 금액 / 총 받은 금액 / 차액 / 이번 달 예정 (4분할, borderless inline)
4. **다가오는 일정 테이블** — 날짜, D-day, 행사, 대상, 관계, 예상 금액
5. **최근 경조사 기록 테이블** — 날짜, 행사, 대상, 관계, 방향, 금액

### 5.2 Stat Bar

- 4분할 inline 통계 (카드 아님)
- 구분: 1px #E5E7EB gap
- 외곽: border-radius 8px, border 1px #E5E7EB
- 각 셀: 라벨(11px #9CA3AF) + 금액(20px SemiBold #111827) + 부가정보(11px)
- 차액이 음수: #DC2626
- 이번 달 예정 금액: Lemon Gold(#EAB308) 11px

### 5.3 기간 필터

- **커스텀 셀렉트:** 브라우저 네이티브 `<select>` 사용 금지, Radix UI Select 기반 커스텀 구현
  - Trigger: border #E5E7EB, radius 6px, padding 7px 12px, chevron-down 아이콘
  - Dropdown: border #E5E7EB, radius 8px, shadow 0 4px 16px rgba(0,0,0,0.08)
  - Item hover: #F9FAFB
  - Item selected: #111827 배경, #fff 텍스트
- **날짜 범위 피커:** 캘린더 아이콘 + 시작일 — 종료일 표시
- 셀렉트 변경 시 날짜 범위 자동 연동, 날짜 직접 선택 시 셀렉트는 "직접 선택"으로 변경

### 5.4 다가오는 일정 테이블

| 컬럼 | 너비 | 설명 |
|------|------|------|
| 날짜 | 140px | yyyy.mm.dd (요일) |
| D-day | 56px | D-N 표기, 임박(3일내) Lemon Gold 강조 |
| 행사 | flex | 결혼식, 돌잔치, 장례식 등 |
| 대상 | 100px | 인물명 |
| 관계 | 80px | pill badge (#F3F4F6 배경) |
| 예상 금액 | 120px | 우측 정렬, #9CA3AF |

### 5.5 최근 경조사 기록 테이블

| 컬럼 | 너비 | 설명 |
|------|------|------|
| 날짜 | 140px | yyyy.mm.dd (요일) |
| 행사 | flex | 행사 유형 |
| 대상 | 100px | 인물명 |
| 관계 | 80px | pill badge |
| 방향 | 80px | "보냄"(#DC2626) / "받음"(#2563EB) |
| 금액 | 120px | 우측 정렬, 방향에 따라 색상 적용 |

---

## 6. 상세 페이지 — 경조사 기록 (webadmin)

### 6.1 페이지 구성

1. **헤더:** 페이지 제목 "경조사 기록" + "새 기록 추가" CTA 버튼(Lemon Gold)
2. **필터바:** 검색(실시간, debounce 300ms) + 행사유형/관계/방향 커스텀 셀렉트 + 날짜 범위 + 총 건수
3. **테이블:** 체크박스 + 날짜 + 행사 + 대상 + 관계 + 방향 + 금액 + ⋮ 액션
4. **페이지네이션:** 현재 페이지 #111827 강조, 좌우 화살표

### 6.2 테이블 컬럼

| 컬럼 | 너비 | 설명 |
|------|------|------|
| 체크박스 | 36px | 다중 선택 |
| 날짜 | 140px | yyyy.mm.dd (요일), 정렬 가능 |
| 행사 | flex | 행사 유형 |
| 대상 | 110px | 인물명 |
| 관계 | 80px | pill badge |
| 방향 | 80px | "보냄" / "받음" |
| 금액 | 120px | 우측 정렬, 정렬 가능, 방향별 색상 |
| 액션 | 48px | ⋮ more 메뉴 |

### 6.3 테이블 인터랙션

- **행 상태:** Default(#fff) / Zebra(#F9FAFB) / Hover(#F3F4F6) / Selected(#FFFBEB)
- **정렬:** 날짜, 금액 컬럼 헤더 클릭 → ASC/DESC 토글, 활성 컬럼 화살표 표시
- **검색:** 이름, 행사명 대상 실시간 필터링
- **필터 셀렉트:** 행사유형(결혼식/돌잔치/장례식/기타), 관계(가족/친구/직장동료/지인), 방향(보냄/받음)
- **다중 선택:** 체크박스 → 하단 "N건 선택됨" + 일괄 삭제
- **행 클릭:** 상세 보기 (체크박스 영역 제외)
- **⋮ 액션 메뉴:** 수정 / 상세 보기 / 삭제(destructive)

### 6.4 페이지네이션

- 현재 페이지: #111827 배경, #fff 텍스트, 32×32px, radius 6px
- 비활성 페이지: #6B7280 텍스트
- 좌우 화살표: chevron 아이콘

---

## 7. 모바일 appuser

### 7.1 홈 화면

1. **Header:** 로고(M + MEMON) + 알림 아이콘
2. **Welcome:** 인사말 + 날짜
3. **Stat Bar:** 3분할 (보낸 금액 / 받은 금액 / 차액), 만원 단위 축약. 이번 달 예정은 모바일 공간 효율을 위해 생략 — 다가오는 일정 섹션에서 확인 가능
4. **다가오는 일정:** 리스트 형태, D-day 우측 표시
5. **최근 기록:** 리스트 형태

### 7.2 기록 탭

1. **Header:** 제목 "경조사 기록" + FAB(Lemon Gold 원형, + 아이콘)
2. **검색 + 필터:** 검색 입력 + 필터 아이콘(bottom sheet)
3. **기간 pill 탭:** 전체 / 이번 달 / 올해 / 작년 (활성: #111827, 비활성: #F3F4F6)
4. **리스트:** 월별 sticky 그룹 헤더 + 행사·대상 1줄 + 날짜·관계 2줄 구조
5. **스와이프:** 좌 스와이프 → 수정/삭제 액션

### 7.3 관계 탭 / 설정 탭

- **관계 탭:** 기록 탭과 동일한 리스트 패턴 적용. 인물별 리스트(이름·관계·총 보낸/받은 금액). 상세 디자인은 별도 spec에서 정의.
- **설정 탭:** 테마 선택(4개 프리뷰), 프로필, 알림 설정. 상세 디자인은 별도 spec에서 정의.

### 7.4 테이블→리스트 변환 규칙

모바일에서는 테이블 대신 리스트 아이템으로 변환:

```
┌─────────────────────────────────────┐
│ 결혼식 · 김철수              -₩50,000│
│ 2026.03.15 (토) · [직장동료]    보냄 │
└─────────────────────────────────────┘
```

- 1줄: 행사 · 대상 (좌) + 금액 (우)
- 2줄: 날짜 (요일) · 관계 badge (좌) + 방향 (우)

### 7.5 모바일 디자인 원칙

- 테이블→리스트 변환 (모바일 최적화)
- 월별 sticky 그룹 헤더로 시간 구분
- 금액 축약: Stat Bar에서만 만원 단위, 리스트는 원 단위
- FAB로 새 기록 추가 (Lemon Gold 원형)
- 좌 스와이프 → 수정/삭제
- 탭 바 4탭, 활성 #111827 / 비활성 #9CA3AF
- 테마 변경: 설정 탭에서 동일하게 제공

---

## 8. 공통 UI 컴포넌트 변경 사항

### 8.1 커스텀 셀렉트 (Select)

- 브라우저 네이티브 `<select>` 사용 금지
- 기존 `src/shared/ui/select.tsx` 컴포넌트를 수정하여 적용 (신규 생성 아님)
- Radix UI Select 기반 커스텀 구현
- Trigger: border #E5E7EB, radius 6px, chevron-down
- Dropdown: border #E5E7EB, radius 8px, shadow 0 4px 16px rgba(0,0,0,0.08)
- Item hover: #F9FAFB
- Item selected: #111827 배경, #fff 텍스트

### 8.2 버튼 (Button)

- **Primary:** `--primary` 배경, `--primary-foreground` 텍스트 (Lemon Gold: #111827 어두운 텍스트, 나머지 테마: #fff)
- **Secondary:** border #E5E7EB, #fff 배경, #111827 텍스트
- **Destructive:** #DC2626 배경, #fff 텍스트
- **Ghost:** 투명, hover #F3F4F6
- 기존 gradient 버튼 제거

### 8.3 체크박스 (Checkbox)

- 미선택: 14×14px, border 1.5px #D1D5DB, radius 3px
- 선택: `--primary` 배경, 흰색 체크 아이콘

### 8.4 관계 Badge

- 배경: #F3F4F6
- 텍스트: inherit (11px)
- radius: 4px
- padding: 2px 8px

### 8.5 페이지네이션 (Pagination)

- 아이템 크기: 32×32px, radius 6px
- 현재 페이지: #111827 배경, #fff
- 기타 페이지: #6B7280

### 8.6 트랜지션 & 애니메이션

- **기본 트랜지션:** 150ms ease-in-out (hover, color, background 변경)
- **사이드바 접기/펼치기:** 200ms ease-in-out
- **드롭다운 열기/닫기:** 150ms ease-out / 100ms ease-in
- **행 hover:** 150ms ease-in-out (background-color)
- **모바일 스와이프 액션:** 200ms ease-out

---

## 9. 테마 시스템 구현

### 9.1 CSS 변수 기반

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

  /* 테마 (변경 가능) */
  --primary: #EAB308;
  --primary-hover: #CA8A04;
  --primary-light: #FEF9C3;
  --primary-foreground: #111827; /* Lemon Gold: 어두운 텍스트 (WCAG 대비 충족) */
  --row-selected: #FFFBEB;
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

### 9.2 Tailwind 컬러 매핑 추가

`preset.js`에 새 CSS 변수 기반 Tailwind 컬러를 추가한다:

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
  primary: 'var(--primary)',
  'primary-hover': 'var(--primary-hover)',
  'primary-light': 'var(--primary-light)',
  'primary-foreground': 'var(--primary-foreground)',
  ring: 'var(--ring)',
  input: 'var(--input)',
  disabled: 'var(--disabled)',
  'disabled-foreground': 'var(--disabled-foreground)',
}
```

### 9.3 테마 저장

- localStorage에 테마 키 저장
- 설정 페이지에서 4가지 테마 프리뷰 + 선택
- 선택 즉시 적용 (페이지 리로드 불필요)
- appuser: AsyncStorage에 동일 구조로 저장

---

## 10. 제거 대상 (기존 디자인 시스템)

| 제거 항목 | 이유 |
|-----------|------|
| Lemon Palette (#D9C85F) 기반 전체 컬러 | 새 Lemon Gold(#EAB308) + White 기반으로 교체 |
| `--brand-deep` (#4C5332) | 사용하지 않음 |
| `--success` (#6E8B4E) | 시맨틱 컬러로 대체 불필요 |
| `--point-navy`, `--point-gray` | 사용하지 않음 |
| `--green10`~`--green100` tints | 새 primary-light로 대체 |
| `.bg-primary-gradient` | 그라데이션 금지 |
| `.border-primary-gradient` | 그라데이션 금지 |
| `.text-primary-gradient` | 그라데이션 금지 |
| KPI 카드 기반 대시보드 | Stat Bar + 테이블로 교체 |
| 차트 컬러 배열 | 차트 사용 최소화, 필요 시 gray 계열만 |
| `--secondary` (#00AFDF) | 사용하지 않음 |
| `index.css` 네이티브 `<select>` 스타일링 (lines ~126-161) | 커스텀 셀렉트로 대체 |
| `index.css` 스크롤바 `var(--point-gray)` 참조 | `var(--placeholder)`로 교체 |
| `index.css` 캘린더 `bg-primary-100`, `bg-primary-10`, hardcoded rgba(217,200,95,0.1) | 새 토큰으로 교체 |

---

## 11. 영향 범위

### 11.1 31_webadmin

| 영역 | 파일/디렉토리 | 변경 내용 |
|------|-------------|-----------|
| 전역 스타일 | `src/index.css` | CSS 변수 전면 교체, 그라데이션 제거 |
| 테마 시스템 | `src/shared/lib/theme.ts` (신규) | 테마 관리 로직 |
| Tailwind 프리셋 | `21_hs-common/src/tailwind/preset.js` | 컬러 토큰 교체 |
| Tailwind 설정 | `tailwind.config.js` | webadmin 전용 컬러 정리 |
| 레이아웃 | `src/widgets/layout/AdminLayout.tsx` | 구조 유지, 스타일 변경 |
| 사이드바 | `src/widgets/sidebar/Sidebar.tsx` | 축소/확장 토글 + 새 컬러 |
| 헤더 | `src/widgets/header/Header.tsx` | 미니멀 헤더로 변경 |
| 대시보드 | `src/widgets/dashboard/*` | 카드→Stat Bar+테이블 전면 교체 |
| 대시보드 페이지 | `src/app/(authenticated)/page.tsx` | 기간 필터 + 테이블 구조 |
| UI 컴포넌트 | `src/shared/ui/*` | 버튼, 셀렉트, 체크박스 variant 교체 |
| variant 파일 | `src/shared/ui/styles/*` | 버튼/배지 등 variant 전면 수정 |
| 설정 페이지 | `src/app/(authenticated)/settings/*` | 테마 선택 UI 추가 |

### 11.2 32_appuser

| 영역 | 파일/디렉토리 | 변경 내용 |
|------|-------------|-----------|
| 전역 스타일 | 스타일 상수 파일 | 컬러 토큰 교체 |
| 테마 시스템 | `src/stores/theme.ts` (신규) | AsyncStorage 기반 테마 관리 (기존 stores/ 디렉토리 활용) |
| 홈 화면 | `app/(tabs)/index.tsx` | Stat Bar + 리스트 구조로 변경 |
| 기록 탭 | `app/(tabs)/records.tsx` | 월별 그룹 리스트 + pill 필터 |
| 관계 탭 | `app/(tabs)/relations.tsx` | 리스트 구조 적용 |
| 설정 탭 | `app/(tabs)/settings.tsx` | 테마 선택 UI 추가 |
| 탭 바 | `app/(tabs)/_layout.tsx` | 아이콘/컬러 변경 |

### 11.3 21_hs-common

| 영역 | 파일 | 변경 내용 |
|------|------|-----------|
| Tailwind 프리셋 | `src/tailwind/preset.js` | 컬러 변수 전면 교체, 그라데이션 유틸리티 제거 |

---
