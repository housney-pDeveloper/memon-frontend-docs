# MEMON 기능 제안 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 13개 추가 기능 제안을 구현하여 MEMON webadmin의 완성도와 사용자 경험을 대폭 향상한다.

**Architecture:** FSD 아키텍처 준수. Recharts(이미 설치됨) 활용 차트 고도화. 기존 API/컴포넌트 패턴 따름.

**Tech Stack:** React 19, TypeScript, TanStack Table/Query, Recharts, Zustand, Tailwind CSS, Spring Boot 3.5, MyBatis, PostgreSQL

**Spec:** `01_docs/docs-claude/specs/2026-03-21-feature-suggestions.md`

**선행 계획:** `01_docs/docs-claude/plans/2026-03-21-ui-improvement-plan.md` 완료 후 수행

---

## Phase 5: Low-Hanging Fruit (제안 5, 7, 3)

### Task 16: 기록 삭제 기능 완성 + 일괄 작업 (제안 5)

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/records/page.tsx`
- Modify: `31_webadmin/src/entities/record/model/` (mutation hook)

- [ ] **Step 1: 단건 삭제 기능 구현**

records/page.tsx의 `// TODO: delete action` 부분에 실제 삭제 로직 추가:

```tsx
const handleDelete = useCallback(async (recordNo: number) => {
  const confirmed = await alert.confirm('정말 삭제하시겠습니까?')
  if (!confirmed) return
  await deleteRecord(recordNo)
  queryClient.invalidateQueries({ queryKey: ['records'] })
  alert.success('삭제되었습니다.')
}, [deleteRecord, queryClient])
```

- [ ] **Step 2: 일괄 삭제 기능 구현**

선택된 행(selectedRows) 기반 일괄 삭제:

```tsx
const handleBatchDelete = useCallback(async () => {
  if (selectedRows.size === 0) return
  const confirmed = await alert.confirm(`${selectedRows.size}건을 삭제하시겠습니까?`)
  if (!confirmed) return
  await Promise.all([...selectedRows].map(recordNo => deleteRecord(recordNo)))
  queryClient.invalidateQueries({ queryKey: ['records'] })
  setSelectedRows(new Set())
  alert.success('삭제되었습니다.')
}, [selectedRows, deleteRecord, queryClient])
```

- [ ] **Step 3: 일괄 삭제 버튼 UI 추가**

선택된 행이 있을 때만 표시되는 "선택 삭제" 버튼을 헤더 영역에 추가.

- [ ] **Step 4: Backend 일괄 삭제 API 확인/추가**

기존 deleteRecord가 단건만 지원하면 `/deleteRecords` (배열) API 추가. 이미 있으면 스킵.

---

### Task 17: 통계 페이지 차트 고도화 (제안 7)

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/statistics/page.tsx`
- Create: `31_webadmin/src/widgets/statistics/MonthlyTrendChart.tsx`
- Create: `31_webadmin/src/widgets/statistics/TypeDistributionChart.tsx`
- Create: `31_webadmin/src/widgets/statistics/TopRelationsChart.tsx`

- [ ] **Step 1: MonthlyTrendChart 위젯 생성**

Recharts `BarChart` + `Line` 복합 차트:
- X축: 월 (YYYY.MM)
- 좌측 Y축: 금액 (보낸/받은 BarChart)
- 우측 Y축: 건수 (Line)
- 범례: 보낸 금액(red), 받은 금액(blue), 건수(gray line)

```tsx
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer, Line, ComposedChart } from 'recharts'

export function MonthlyTrendChart({ data }: { data: MonthlyTrend[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <ComposedChart data={data}>
        <CartesianGrid strokeDasharray="3 3" />
        <XAxis dataKey="month" />
        <YAxis yAxisId="left" tickFormatter={(v) => `${(v/10000).toFixed(0)}만`} />
        <YAxis yAxisId="right" orientation="right" />
        <Tooltip />
        <Legend />
        <Bar yAxisId="left" dataKey="totalGivenAmount" name="보낸 금액" fill="#DC2626" />
        <Bar yAxisId="left" dataKey="totalReceivedAmount" name="받은 금액" fill="#2563EB" />
        <Line yAxisId="right" dataKey="totalCount" name="건수" stroke="#6B7280" />
      </ComposedChart>
    </ResponsiveContainer>
  )
}
```

- [ ] **Step 2: TypeDistributionChart 위젯 생성**

Recharts `PieChart`:
- 기록 유형별 분포 (WEDDING, FUNERAL, BIRTHDAY 등)
- 금액 기반 비율 + 건수 라벨

```tsx
import { PieChart, Pie, Cell, ResponsiveContainer, Tooltip, Legend } from 'recharts'

const COLORS = ['#EAB308', '#DC2626', '#2563EB', '#059669', '#6B7280']

export function TypeDistributionChart({ data }: { data: TypeDistribution[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <PieChart>
        <Pie data={data} dataKey="totalAmount" nameKey="typeName" cx="50%" cy="50%" outerRadius={100} label>
          {data.map((_, i) => <Cell key={i} fill={COLORS[i % COLORS.length]} />)}
        </Pie>
        <Tooltip formatter={(v: number) => `${v.toLocaleString()}원`} />
        <Legend />
      </PieChart>
    </ResponsiveContainer>
  )
}
```

- [ ] **Step 3: TopRelationsChart 위젯 생성**

Recharts 수평 `BarChart`:
- 상위 10개 관계별 거래 금액/건수

- [ ] **Step 4: statistics/page.tsx에서 기존 div 바를 차트 위젯으로 교체**

기존 `width: ${percent}%` 스타일 기반 바 차트를 Recharts 위젯으로 교체.

---

### Task 18: 캘린더 그리드 뷰 (제안 3)

**Files:**
- Create: `31_webadmin/src/widgets/schedule/CalendarGrid.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/schedules/page.tsx`

- [ ] **Step 1: CalendarGrid 컴포넌트 생성**

기존 `react-day-picker` (Calendar 컴포넌트에서 사용 중)를 활용한 월간 캘린더 그리드:

```tsx
import { DayPicker, type DayProps } from 'react-day-picker'
import { ko } from 'date-fns/locale'

interface CalendarGridProps {
  schedules: CalendarSchedule[]
  month: Date
  onMonthChange: (month: Date) => void
  onDayClick: (date: Date) => void
}

export function CalendarGrid({ schedules, month, onMonthChange, onDayClick }: CalendarGridProps) {
  // 날짜별 일정 매핑
  const schedulesByDate = useMemo(() => {
    const map = new Map<string, CalendarSchedule[]>()
    schedules.forEach(s => {
      const key = s.scheduleDate // YYYYMMDD
      map.set(key, [...(map.get(key) ?? []), s])
    })
    return map
  }, [schedules])

  return (
    <DayPicker
      mode="single"
      locale={ko}
      month={month}
      onMonthChange={onMonthChange}
      onDayClick={onDayClick}
      components={{
        Day: (props: DayProps) => {
          const dateStr = format(props.date, 'yyyyMMdd')
          const daySchedules = schedulesByDate.get(dateStr) ?? []
          return (
            <td {...props}>
              <div className="h-80 p-4 border border-border hover:bg-muted/30 cursor-pointer">
                <span className="text-body-5">{props.date.getDate()}</span>
                <div className="mt-4 space-y-2">
                  {daySchedules.slice(0, 3).map((s, i) => (
                    <div key={i} className={cn(
                      'text-[10px] truncate px-4 py-2 rounded',
                      s.scheduleTypeCode === 'BIRTHDAY' && 'bg-pink-100 text-pink-700',
                      s.scheduleTypeCode === 'ANNIVERSARY' && 'bg-blue-100 text-blue-700',
                      s.scheduleTypeCode === 'REMINDER' && 'bg-amber-100 text-amber-700',
                    )}>
                      {s.title}
                    </div>
                  ))}
                  {daySchedules.length > 3 && (
                    <span className="text-[10px] text-gray-3">+{daySchedules.length - 3}</span>
                  )}
                </div>
              </div>
            </td>
          )
        }
      }}
    />
  )
}
```

- [ ] **Step 2: schedules/page.tsx에 그리드 뷰 옵션 추가**

기존 "list" 뷰 옵션에 "grid" 뷰 추가:
- 탭: 다가오는 일정 | 캘린더(리스트) | 캘린더(그리드)
- 그리드 뷰 선택 시 `CalendarGrid` 렌더링

- [ ] **Step 3: 날짜 클릭 시 해당일 일정 Dialog 표시**

클릭한 날짜의 일정 목록을 Dialog로 표시.

---

## Phase 6: Medium Features (제안 1, 12, 6, 13)

### Task 19: 보낸/받은 금액 균형 시각화 (제안 1)

**Files:**
- Create: `31_webadmin/src/widgets/dashboard/BalanceWidget.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/page.tsx` (대시보드에 위젯 추가)
- Backend: 균형 데이터 집계 API 필요시 추가

- [ ] **Step 1: Backend API 확인**

기존 `getHomeSummary()`의 응답에 관계별 보낸/받은 금액 데이터가 있는지 확인. 없으면 `getBalanceTop` API 추가:

```java
@PostMapping("/getBalanceTop")
public ServerResponse<List<BalanceTopResponseDtoV1>> getBalanceTop(@RequestBody PageRequest request) {
  // 관계별 총 보낸/받은 금액 차이 상위 N건 반환
}
```

- [ ] **Step 2: BalanceWidget 컴포넌트 생성**

Recharts 수평 BarChart로 관계별 보낸/받은 금액 균형 시각화:

```tsx
// 양방향 수평 바: 왼쪽(보낸, red), 오른쪽(받은, blue)
// 차액이 큰 순으로 Top 5 표시
```

- [ ] **Step 3: 대시보드 페이지에 위젯 추가**

기존 대시보드 레이아웃에 BalanceWidget 배치.

---

### Task 20: 다크 모드 지원 (제안 12)

**Files:**
- Modify: `31_webadmin/src/index.css`
- Modify: `31_webadmin/src/features/auth/ui/MyPageModal.tsx` (테마 설정에 다크모드 옵션)

- [ ] **Step 1: index.css에 다크 모드 CSS 변수 추가**

각 테마에 대해 `.dark` 변형 추가:

```css
.dark {
  --background: #111827;
  --surface: #1F2937;
  --foreground: #F9FAFB;
  --muted-foreground: #9CA3AF;
  --placeholder: #6B7280;
  --border: #374151;
  --border-light: #1F2937;
  --row-hover: #1F2937;
  --row-zebra: #111827;
  --input: #374151;
  --disabled: #1F2937;
  --disabled-foreground: #6B7280;
  /* primary 색상은 테마별로 유지 */
}

[data-theme="deep-blue"].dark { /* deep-blue 다크 변형 */ }
[data-theme="emerald"].dark { /* emerald 다크 변형 */ }
[data-theme="monochrome"].dark { /* monochrome 다크 변형 */ }
```

- [ ] **Step 2: 테마 설정에 라이트/다크 토글 추가**

MyPageModal의 테마 섹션에:
- 라이트 모드 / 다크 모드 / 시스템 설정 따르기 3가지 옵션
- 선택 시 `document.documentElement.classList.toggle('dark')` + localStorage 저장

- [ ] **Step 3: 시스템 설정 감지**

```ts
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)')
prefersDark.addEventListener('change', (e) => {
  if (themeMode === 'system') {
    document.documentElement.classList.toggle('dark', e.matches)
  }
})
```

---

### Task 21: 데이터 내보내기 (제안 6)

**Files:**
- Create: `31_webadmin/src/features/record/ui/ExportDialog.tsx`
- Create: `31_webadmin/src/shared/lib/export-utils.ts`
- Modify: `31_webadmin/src/app/(authenticated)/records/page.tsx` (내보내기 버튼)

- [ ] **Step 1: export-utils.ts 유틸리티 생성**

CSV/Excel 내보내기 유틸리티:

```ts
export function exportToCSV(data: Record<string, unknown>[], columns: { field: string; title: string }[], filename: string) {
  const header = columns.map(c => c.title).join(',')
  const rows = data.map(row => columns.map(c => `"${row[c.field] ?? ''}"`).join(','))
  const csv = [header, ...rows].join('\n')
  const blob = new Blob(['\uFEFF' + csv], { type: 'text/csv;charset=utf-8' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${filename}.csv`
  a.click()
  URL.revokeObjectURL(url)
}
```

- [ ] **Step 2: ExportDialog 컴포넌트 생성**

내보내기 옵션:
- 형식: CSV / Excel
- 범위: 현재 필터 결과 / 전체 / 선택된 행
- 포함 컬럼 선택

- [ ] **Step 3: records/page.tsx에 내보내기 버튼 추가**

헤더 영역에 "내보내기" 버튼 추가 → ExportDialog 열기.

---

### Task 22: 경조사 요약 리포트 (제안 13)

**Files:**
- Create: `31_webadmin/src/widgets/dashboard/MonthlySummaryCard.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/page.tsx`
- Backend: 요약 API 추가 (필요시)

- [ ] **Step 1: MonthlySummaryCard 컴포넌트**

이번 달 경조사 요약 카드:
- 이번 달 총 지출/수입
- 지난달 대비 변화량 (▲/▼ 표시)
- 가장 많이 교류한 관계

- [ ] **Step 2: 대시보드에 추가**

StatBar 아래에 MonthlySummaryCard 배치.

---

## Phase 7: Advanced Features (제안 10, 9, 11, 8, 4, 2)

### Task 23: 전역 검색 (제안 10)

**Files:**
- Create: `31_webadmin/src/widgets/header/GlobalSearch.tsx`
- Modify: `31_webadmin/src/widgets/header/Header.tsx`
- Backend: 통합 검색 API 추가

- [ ] **Step 1: Backend 통합 검색 API**

```java
@PostMapping("/search")
public ServerResponse<GlobalSearchResponseDtoV1> globalSearch(@RequestBody GlobalSearchRequestDtoV1 dto) {
  // 관계, 기록, 경조사, 일정에서 keyword로 검색
  // 각 카테고리별 상위 5건 반환
}
```

- [ ] **Step 2: GlobalSearch 컴포넌트**

Command+K 스타일 검색 팝오버:
- 입력 시 debounce 검색
- 카테고리별 결과 그룹핑
- 클릭 시 해당 페이지로 이동

- [ ] **Step 3: Header에 검색 아이콘 + GlobalSearch 추가**

---

### Task 24: 관계 프로필 아바타 (제안 9)

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/relations/page.tsx`
- Create: `31_webadmin/src/shared/ui/relationship-avatar.tsx`
- Backend: 이미지 업로드 API (선택)

- [ ] **Step 1: RelationshipAvatar 컴포넌트 생성**

이니셜 기반 아바타 (관계 유형 색상 자동 지정):

```tsx
export function RelationshipAvatar({ name, typeColor, size = 'md' }: Props) {
  const initial = name.charAt(0)
  return (
    <div className={cn(
      'rounded-full flex items-center justify-center text-white font-semibold',
      size === 'sm' && 'h-32 w-32 text-body-5',
      size === 'md' && 'h-40 w-40 text-body-3',
    )} style={{ backgroundColor: typeColor || '#6B7280' }}>
      {initial}
    </div>
  )
}
```

- [ ] **Step 2: 관계 카드에 아바타 추가**

기존 관계 카드의 이름 좌측에 RelationshipAvatar 배치.

---

### Task 25: 경조사 중복 방지 및 연결 (제안 11)

**Files:**
- Modify: `31_webadmin/src/features/ceremony/ui/CeremonyForm.tsx`
- Backend: 유사 경조사 검색 API

- [ ] **Step 1: Backend 유사 경조사 검색 API**

```java
@PostMapping("/searchSimilar")
public ServerResponse<List<CeremonyListResponseDtoV1>> searchSimilar(@RequestBody SearchSimilarRequestDtoV1 dto) {
  // ceremonyTypeCode + ceremonyDate 기준 ±7일 범위 내 유사 경조사 검색
}
```

- [ ] **Step 2: CeremonyForm에 유사 경조사 자동 제안**

날짜/유형 선택 시 유사 경조사가 있으면 "기존 경조사에 연결하시겠습니까?" 안내 표시.

---

### Task 26: 알림 설정 (제안 8)

**Files:**
- Database: 알림 설정 테이블 추가
- Backend: 알림 설정 CRUD API + 스케줄러
- Frontend: MyPageModal 또는 별도 설정 UI

- [ ] **Step 1: Database 테이블 생성**

```sql
CREATE TABLE scm_mmhr.tb_notification_setting (
    setting_no      BIGSERIAL PRIMARY KEY,
    user_no         BIGINT NOT NULL,
    target_type     VARCHAR(20) NOT NULL, -- BIRTHDAY / ANNIVERSARY / SCHEDULE
    days_before     INT NOT NULL DEFAULT 1,
    channel_code    VARCHAR(20) NOT NULL DEFAULT 'IN_APP', -- IN_APP / EMAIL / PUSH
    enabled_yn      CHAR(1) DEFAULT 'Y',
    reg_date        TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    mod_date        TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

- [ ] **Step 2: Backend CRUD API**

```
POST /notification-setting/getList
POST /notification-setting/save
POST /notification-setting/delete
```

- [ ] **Step 3: System Server 스케줄러**

매일 실행되는 배치 작업: 알림 설정에 따라 D-day 전 알림 생성.

- [ ] **Step 4: Frontend 알림 설정 UI**

관계 상세 페이지 또는 MyPageModal에서 알림 설정 관리.

---

### Task 27: 일괄 기록 등록 / Excel 가져오기 (제안 4)

**Files:**
- Create: `31_webadmin/src/features/record/ui/BulkImportDialog.tsx`
- Create: `31_webadmin/src/shared/lib/csv-parser.ts`
- Backend: 일괄 등록 API

- [ ] **Step 1: Backend 일괄 등록 API**

```java
@PostMapping("/createRecords")
public ServerResponse<Integer> createRecords(@RequestBody @Valid List<CreateRecordRequestDtoV1> dtos) {
  // 배열로 받아 bulk insert
  // 반환: 등록 성공 건수
}
```

- [ ] **Step 2: CSV 파서 유틸리티**

```ts
export function parseCSV(content: string): Record<string, string>[] {
  const lines = content.split('\n')
  const headers = lines[0].split(',').map(h => h.trim().replace(/"/g, ''))
  return lines.slice(1).filter(l => l.trim()).map(line => {
    const values = line.split(',').map(v => v.trim().replace(/"/g, ''))
    return Object.fromEntries(headers.map((h, i) => [h, values[i] ?? '']))
  })
}
```

- [ ] **Step 3: BulkImportDialog 컴포넌트**

- CSV/Excel 파일 업로드 (드래그 앤 드롭)
- 미리보기 테이블 (DataGrid)
- 컬럼 매핑 (파일 컬럼 → DB 필드)
- "등록" 버튼 → API 호출

---

### Task 28: 금액 추천 엔진 (제안 2)

**Files:**
- Backend: 추천 로직 API
- Frontend: RecordForm 내 추천 UI

- [ ] **Step 1: Backend 추천 API**

```java
@PostMapping("/getRecommendedAmount")
public ServerResponse<AmountRecommendationResponseDtoV1> getRecommendedAmount(
    @RequestBody AmountRecommendationRequestDtoV1 dto) {
  // 1. 해당 관계에서 과거 받은 금액 조회
  // 2. 같은 유형 경조사의 전체 평균 금액
  // 3. 관계 유형별 평균 금액
  // 반환: { pastReceivedAmount, typeAverage, relationAverage, recommended }
}
```

- [ ] **Step 2: RecordForm에 금액 추천 표시**

금액 필드 아래에 추천 금액 표시:

```tsx
{recommendation && (
  <div className="text-body-7 text-gray-3 mt-4">
    <p>추천: {recommendation.recommended.toLocaleString()}원</p>
    <p className="text-body-8">과거 받은 금액: {recommendation.pastReceivedAmount.toLocaleString()}원</p>
  </div>
)}
```

---

## 검증

- [ ] **전체 빌드**: `cd 31_webadmin && npm run build`
- [ ] **Backend 빌드**: `cd 32_application && ./gradlew build`
- [ ] **Type check**: `cd 31_webadmin && npm run type-check`
- [ ] **Lint**: `cd 31_webadmin && npm run lint`
