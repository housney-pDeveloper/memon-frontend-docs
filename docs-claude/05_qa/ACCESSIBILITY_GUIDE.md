# 접근성 가이드

> 프론트엔드 프로젝트(webadmin, appuser) 전체에 적용되는 접근성 가이드.

---

## 1. 기준

**WCAG 2.1 AA** 준수

## 2. 시맨틱 HTML

- `<nav>`, `<main>`, `<article>`, `<aside>`, `<section>` 적극 사용
- 제목 계층: `<h1>` → `<h2>` → `<h3>` (건너뛰기 금지)
- 폼: `<label>` + `htmlFor` 또는 중첩
- 버튼: `<button>`, 링크: `<a>` (역할 혼용 금지)
- 리스트: `<ul>`/`<ol>` + `<li>`

## 3. ARIA

- 시맨틱 태그로 해결 불가 시에만 ARIA 사용
- `aria-label` / `aria-labelledby`: 비시각 정보 제공
- `aria-live="polite"`: 동적 콘텐츠 (토스트, 알림)
- `role="dialog"` + `aria-modal="true"`: 모달
- `aria-expanded`: 드롭다운, 아코디언
- `aria-invalid` + `aria-describedby`: 폼 에러

## 4. 키보드 접근성

- **Tab**: 모든 인터랙티브 요소 도달 가능
- **Enter/Space**: 버튼/체크박스 활성화
- **Escape**: 모달/드롭다운 닫기
- **화살표**: 라디오/탭/메뉴 탐색
- **포커스 트랩**: 모달 열림 시 포커스 범위 제한
- **포커스 인디케이터**: `focus-visible:ring-2` (Tailwind)

## 5. 색상 대비

- 일반 텍스트: 4.5:1 이상
- 큰 텍스트(18px+): 3:1 이상
- 색상만으로 정보 전달 금지 (아이콘/텍스트 병행)

## 6. 모바일 접근성 (appuser)

- 터치 타겟: 최소 44x44px
- 핀치 줌 차단 금지 (`user-scalable=no` 사용 금지)
- 가로/세로 모드 지원
- `alt` 텍스트 필수

## 7. 컴포넌트별 접근성 기준

| 컴포넌트 | 필수 접근성 속성 |
|----------|-----------------|
| Button | `role="button"`, 비활성 시 `aria-disabled` |
| Input | `aria-invalid`, `aria-describedby` (에러 메시지) |
| Dialog/Popup | 포커스 트랩, Escape 닫기, `aria-modal` |
| DataGrid | `role="grid"`, 셀 탐색, 정렬 상태 |
| Tabs | `role="tablist"`, 화살표 키 전환 |
| Toast | `aria-live="polite"` |
| AutoComplete | `role="combobox"`, `aria-expanded` |
| Sidebar | `role="navigation"`, 현재 위치 표시 |
