# 퍼블리싱 코드 컨벤션

> 11_publishing 프로젝트의 코드 작성 규칙

---

## 1. 프로젝트 성격

- **UI 참조/데모용 프로젝트** — 비즈니스 로직 포함 금지
- 디자인 시안을 HTML/Tailwind CSS 마크업으로 변환하는 것이 목적
- webadmin/appuser 개발 시 **시각적 참조**로 활용
- 실제 API 연동 없음 — **목업 데이터**만 사용

---

## 2. 프로젝트 정보

| 항목 | 값 |
|------|-----|
| 경로 | `11_publishing/` |
| 포트 | `8000` |
| 프레임워크 | Next.js 15 |
| UI 라이브러리 | Radix UI, shadcn/ui, Embla Carousel, Recharts, Vaul |

---

## 3. 파일 명명 규칙

| 유형 | 규칙 | 예시 |
|------|------|------|
| 컴포넌트 | `PascalCase.tsx` | `SectionHeader.tsx`, `UserCard.tsx` |
| 유틸리티 | `camelCase.ts` | `formatDate.ts`, `cn.ts` |
| 페이지 | Next.js App Router 규칙 | `page.tsx`, `layout.tsx` |

---

## 4. 컴포넌트 작성 규칙

```tsx
interface SectionHeaderProps {
  title: string
  description?: string
}

export const SectionHeader = ({ title, description }: SectionHeaderProps) => {
  return (
    <div className="mb-8">
      <h2 className="text-title-4 font-bold text-gray-900">{title}</h2>
      {description && <p className="mt-2 text-body-3 text-gray-500">{description}</p>}
    </div>
  )
}
SectionHeader.displayName = "SectionHeader"
```

### 필수 규칙

- **Arrow Function** 사용 + **displayName** 필수 설정
- **Props 인터페이스** 분리 선언 (컴포넌트 상단에 위치)

---

## 5. 핵심 원칙

### 비즈니스 로직 금지

- 상태 관리, API 호출, 인증 등 포함하지 않음
- 퍼블리싱 프로젝트는 순수 UI 마크업만 담당

### 목업 데이터만 사용

- 하드코딩된 샘플 데이터 사용
- 실제 API 응답 구조를 참고하되 직접 호출하지 않음

### 시맨틱 HTML

- 적절한 태그 사용: `<nav>`, `<main>`, `<section>`, `<article>`
- 의미에 맞는 HTML 요소 선택

### 반응형 필수

- 모바일 / 태블릿 / 데스크톱 브레이크포인트 대응
- 모바일 퍼스트 접근 방식

### hs-common 토큰 사용

- 색상, 타이포그래피, 간격은 디자인 토큰 활용
- 직접 색상 값 하드코딩 금지

---

## 6. 금지 사항

| 금지 항목 | 설명 |
|-----------|------|
| API 호출 금지 | `axios`, `fetch` 등 사용 불가 |
| 상태 관리 라이브러리 사용 금지 | Zustand, React Query 등 사용 불가 |
| 인증/인가 로직 금지 | 로그인, 토큰 관리 등 포함 불가 |
| 라우트 가드 금지 | 접근 제어 로직 포함 불가 |
| 환경 변수 참조 금지 | `.env` 파일 참조 불가 |
