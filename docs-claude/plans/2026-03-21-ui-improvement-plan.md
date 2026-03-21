# MEMON UI 개선 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 12개 UI/기능 요구사항을 구현하여 MEMON webadmin의 사용성과 일관성을 개선한다.

**Architecture:** FSD 아키텍처(app → widgets → features → entities → shared) 준수. hs-common 변경 시 webadmin 빌드 확인 필수. MyPageModal은 Zustand store 기반으로 app 레이어에서 렌더링.

**Tech Stack:** React 19, TypeScript, TanStack Table, Zustand, React Hook Form + Zod, Tailwind CSS, Radix UI, Spring Boot 3.5 (Java 21), MyBatis, PostgreSQL

**Spec:** `01_docs/docs-claude/specs/2026-03-21-ui-improvement-design.md`

---

## Phase 1: UI 버그 수정 + CSS 개선 (병렬 가능)

### Task 1: 로딩 progress 크기 2배 (요구사항 2)

**Files:**
- Modify: `31_webadmin/src/shared/ui/loading-spinner.tsx`

- [ ] **Step 1: LoadingSpinner sizeMap을 2배로 변경**

```tsx
// loading-spinner.tsx — sizeMap만 변경
const sizeMap = {
  sm: 'h-8 w-8',
  md: 'h-16 w-16',
  lg: 'h-24 w-24',
}
```

- [ ] **Step 2: 전체 사용처 확인 및 조정**

LoadingSpinner를 사용하는 모든 곳을 grep하여 size prop이 적절한지 확인. 인라인 로딩(예: 버튼 내부 스피너)에 sm을 쓰는 곳이 있으면 그대로 유지하되, 페이지 로딩에는 lg 사용.

---

### Task 2: Modal title 글자색 수정 (요구사항 3)

**Files:**
- Modify: `31_webadmin/src/index.css`
- Modify: `21_hs-common/src/tailwind/preset.js`

- [ ] **Step 1: index.css에 --brand-deep CSS 변수 추가**

`index.css`의 `:root` 블록과 각 테마 블록에 추가:

```css
/* :root (lemon-gold 기본) */
--brand-deep: #92400E;

/* [data-theme="deep-blue"] */
--brand-deep: #1E3A5F;

/* [data-theme="emerald"] */
--brand-deep: #064E3B;

/* [data-theme="monochrome"] */
--brand-deep: #111827;
```

- [ ] **Step 2: hs-common preset에 brand-deep 색상 추가**

`21_hs-common/src/tailwind/preset.js`의 `theme.extend.colors`에 추가:

```js
'brand-deep': 'var(--brand-deep)',
```

- [ ] **Step 3: webadmin 빌드 확인**

Run: `cd 31_webadmin && npm run type-check`

---

### Task 3: 로딩 UI 통일 (요구사항 11)

**Files:**
- Modify: `31_webadmin/src/app/(authenticated)/records/page.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/ceremonies/page.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/relations/page.tsx`
- Modify: `31_webadmin/src/shared/components/data-grid.tsx`
- Modify: `31_webadmin/src/widgets/dashboard/UpcomingSection.tsx`
- Modify: `31_webadmin/src/widgets/dashboard/RecentRecordsSection.tsx`

- [ ] **Step 1: records/page.tsx 로딩 상태 변경**

기존 `"불러오는 중..."` 텍스트를 LoadingSpinner로 교체:

```tsx
import { LoadingSpinner } from '@/shared/ui/loading-spinner'

// isLoading 분기 내부:
<div className="flex justify-center items-center py-80">
  <LoadingSpinner size="lg" message="데이터를 불러오는 중..." />
</div>
```

- [ ] **Step 2: ceremonies/page.tsx 동일 변경**

같은 패턴으로 `"불러오는 중..."` → `<LoadingSpinner size="lg" message="데이터를 불러오는 중..." />`

- [ ] **Step 3: relations/page.tsx에 message 추가**

이미 LoadingSpinner를 사용하므로 message prop만 추가:

```tsx
<LoadingSpinner size="lg" message="데이터를 불러오는 중..." />
```

- [ ] **Step 4: data-grid.tsx 로딩 상태 변경**

```tsx
import { LoadingSpinner } from '@/shared/ui/loading-spinner'

// isLoading 분기 내부:
if (isLoading) {
  return (
    <div className="flex h-64 items-center justify-center rounded-lg border border-input">
      <LoadingSpinner size="lg" message="데이터를 불러오는 중..." />
    </div>
  )
}
```

- [ ] **Step 5: UpcomingSection.tsx, RecentRecordsSection.tsx 동일 변경**

각 섹션의 `"불러오는 중..."` 텍스트를 LoadingSpinner로 교체.

---

## Phase 2: Input/Grid 개선

### Task 4: 금액 Input 커스텀 스테퍼 (요구사항 4, 5)

**Files:**
- Modify: `31_webadmin/src/shared/ui/input-money.tsx`
- Modify: `31_webadmin/src/shared/hooks/useNumberInput.ts`
- Modify: `31_webadmin/src/index.css`

- [ ] **Step 1: index.css에 브라우저 기본 spin button 숨김 CSS 추가**

```css
/* 브라우저 기본 number spin button 숨김 */
input[type='number']::-webkit-outer-spin-button,
input[type='number']::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
input[type='number'] {
  -moz-appearance: textfield;
}
```

- [ ] **Step 2: useNumberInput에 increment/decrement 함수 추가**

`useNumberInput.ts`의 반환값에 추가:

```ts
const increment = useCallback((step: number) => {
  const raw = parseCurrency(displayValue)
  const current = raw === '' ? 0 : parseFloat(raw)
  const next = current + step
  const clamped = max !== undefined ? Math.min(next, max) : next
  const formatted = formatCurrency(String(clamped))
  setDisplayValue(formatted)
  onChange(String(clamped))
}, [displayValue, max, onChange])

const decrement = useCallback((step: number) => {
  const raw = parseCurrency(displayValue)
  const current = raw === '' ? 0 : parseFloat(raw)
  const next = current - step
  const clamped = min !== undefined ? Math.max(next, min) : Math.max(next, 0)
  const formatted = formatCurrency(String(clamped))
  setDisplayValue(formatted)
  onChange(String(clamped))
}, [displayValue, min, onChange])

return { ..., increment, decrement }
```

- [ ] **Step 3: InputMoney에 커스텀 스테퍼 UI 추가**

```tsx
export interface InputMoneyProps
  extends Omit<InputProps, 'type' | 'onChange' | 'value'> {
  value?: string
  onChange: (value: string) => void
  min?: number
  max?: number
  stepper?: boolean  // 추가
  step?: number      // 추가
}

const InputMoney = React.memo(
  React.forwardRef<HTMLInputElement, InputMoneyProps>(
    ({ value, onChange, min, max, stepper = true, step = 10000, onFocus, onBlur, className, ...props }, ref) => {
      const {
        inputRef, displayValue, handleChange, handleBlur, handleFocus,
        pattern, inputMode, increment, decrement,
      } = useNumberInput({
        type: 'money', value: value ?? '', onChange, min, max, onFocus, onBlur,
      })

      React.useImperativeHandle(ref, () => inputRef.current as HTMLInputElement)

      return (
        <div className="relative">
          <Input
            {...props}
            ref={inputRef}
            type="text"
            inputMode={inputMode}
            pattern={pattern}
            value={displayValue}
            onChange={handleChange}
            onFocus={handleFocus}
            onBlur={handleBlur}
            className={cn(stepper && 'pr-32', className)}
          />
          {stepper && (
            <div className="absolute right-1 top-1 bottom-1 flex flex-col w-24 border-l border-input">
              <button
                type="button"
                tabIndex={-1}
                className="flex-1 flex items-center justify-center hover:bg-muted/50 text-gray-3 transition-colors"
                onClick={() => increment(step)}
              >
                <ChevronUp className="h-12 w-12" />
              </button>
              <div className="border-t border-input" />
              <button
                type="button"
                tabIndex={-1}
                className="flex-1 flex items-center justify-center hover:bg-muted/50 text-gray-3 transition-colors"
                onClick={() => decrement(step)}
              >
                <ChevronDown className="h-12 w-12" />
              </button>
            </div>
          )}
        </div>
      )
    }
  )
)
```

Import 추가: `import { ChevronUp, ChevronDown } from 'lucide-react'`

- [ ] **Step 4: 빌드 확인**

Run: `cd 31_webadmin && npm run type-check`

---

### Task 5: Grid 컬럼 너비 조정 (요구사항 1)

**Files:**
- Modify: `31_webadmin/src/shared/components/data-grid.tsx`

- [ ] **Step 1: DataGridColumn에 minWidth prop 추가**

```ts
export interface DataGridColumn<T extends object = Record<string, unknown>> {
  field: string
  title?: string
  width?: number | string
  minWidth?: number  // 추가
  // ... 기존 props 유지
}
```

- [ ] **Step 2: useReactTable에 column resizing 활성화**

```ts
const table = useReactTable({
  data: gridData as Record<string, unknown>[],
  columns: columnDefs,
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  enableColumnResizing: true,
  columnResizeMode: 'onChange',
  // ... 기존 설정 유지
})
```

- [ ] **Step 3: columnDefs에 minSize 설정 추가**

```ts
defs.push({
  id: col.field,
  accessorKey: col.field,
  size: typeof col.width === 'number' ? col.width : 120,
  minSize: col.minWidth ?? 80,
  enableSorting: col.sortable ?? sortable,
  enableResizing: true,
  // ... 기존 설정 유지
})
```

- [ ] **Step 4: table-fixed 제거 및 리사이즈 핸들 CSS 추가**

`table-fixed` → 제거. 헤더 `<th>`에 리사이즈 핸들 추가:

```tsx
<th key={header.id} style={{ width: header.getSize(), position: 'relative', ... }}>
  {/* 기존 헤더 내용 */}
  {header.column.getCanResize() && (
    <div
      onMouseDown={header.getResizeHandler()}
      onTouchStart={header.getResizeHandler()}
      className={cn(
        'absolute right-0 top-0 h-full w-4 cursor-col-resize select-none touch-none',
        'hover:bg-primary/20',
        header.column.getIsResizing() && 'bg-primary/30'
      )}
    />
  )}
</th>
```

- [ ] **Step 5: 빌드 확인**

Run: `cd 31_webadmin && npm run type-check`

---

## Phase 3: UI 구조 변경

### Task 6: MyPage 버튼 + Store (요구사항 8)

**Files:**
- Create: `31_webadmin/src/shared/stores/myPageStore.ts`
- Modify: `31_webadmin/src/widgets/header/Header.tsx`
- Modify: `31_webadmin/src/shared/config/menu-config.ts`

- [ ] **Step 1: myPageStore.ts 생성**

```ts
import { create } from 'zustand'

interface MyPageStore {
  isOpen: boolean
  open: () => void
  close: () => void
}

export const useMyPageStore = create<MyPageStore>((set) => ({
  isOpen: false,
  open: () => set({ isOpen: true }),
  close: () => set({ isOpen: false }),
}))
```

- [ ] **Step 2: Header.tsx에서 아바타를 마이페이지 버튼으로 변경**

기존 아바타 원형(성 첫글자)을 User 아이콘 버튼으로 교체:

```tsx
import { User } from 'lucide-react'
import { useMyPageStore } from '@/shared/stores/myPageStore'

// 기존 아바타 부분 교체:
<button
  onClick={useMyPageStore.getState().open}
  className="flex items-center gap-4 px-8 py-4 rounded-md hover:bg-muted transition-colors"
  title="마이페이지"
>
  <User className="h-20 w-20 text-gray-3" />
  <span className="text-body-5 text-gray-3">마이페이지</span>
</button>
```

- [ ] **Step 3: menu-config.ts에서 설정 메뉴 제거**

`sidebarMenuItems` 배열에서 `id: 'settings'` 항목을 제거.

---

### Task 7: MyPageModal 컴포넌트 (요구사항 8, 9)

**Files:**
- Create: `31_webadmin/src/features/auth/ui/MyPageModal.tsx`
- Modify: `31_webadmin/src/widgets/layout/AdminLayout.tsx`
- Modify: `31_webadmin/src/app/router/` (settings 라우트 제거)

- [ ] **Step 1: MyPageModal.tsx 생성**

기존 settings/page.tsx의 기능을 dialog로 재구성:
- 내 정보 (이름 편집 가능 + 저장)
- 비밀번호 변경 (현재/신규/확인)
- 테마 선택 (기존 4개 테마)
- 로그아웃

```tsx
import { useMyPageStore } from '@/shared/stores/myPageStore'
import { Dialog, DialogContent, DialogHeader, DialogBody, DialogTitle } from '@/shared/ui/dialog'
import { useAuthStore } from '@/entities/auth/model/store'
// Zod 스키마, useForm, 등 import

export function MyPageModal() {
  const { isOpen, close } = useMyPageStore()
  const user = useAuthStore((s) => s.user)

  return (
    <Dialog open={isOpen} onOpenChange={(open) => !open && close()}>
      <DialogContent className="max-w-lg">
        <DialogHeader>
          <DialogTitle>마이페이지</DialogTitle>
        </DialogHeader>
        <DialogBody>
          {/* 내 정보 섹션 */}
          {/* 비밀번호 변경 섹션 */}
          {/* 테마 변경 섹션 */}
          {/* 로그아웃 버튼 */}
        </DialogBody>
      </DialogContent>
    </Dialog>
  )
}
```

- [ ] **Step 2: 내 정보 섹션 구현**

이름 수정 폼 (React Hook Form + Zod):
- 이름 Input (기본값: user.userName)
- "저장" 버튼 → `POST /user/updateMyInfo` API 호출
- 성공 시 authStore 업데이트

- [ ] **Step 3: 비밀번호 변경 섹션 구현**

비밀번호 변경 폼:
- 현재 비밀번호 (InputPassword)
- 새 비밀번호 (InputPassword + strength indicator)
- 비밀번호 확인 (InputPassword)
- Zod: `newPassword === confirmPassword` refine 검증
- "변경" 버튼 → `POST /user/changePassword` API 호출

- [ ] **Step 4: 테마 변경 섹션 구현**

기존 settings/page.tsx의 테마 선택 UI를 그대로 이식:
- 4개 테마 카드 (lemon-gold, deep-blue, emerald, monochrome)
- 클릭 시 `document.documentElement.setAttribute('data-theme', theme)`
- localStorage 저장

- [ ] **Step 5: 로그아웃 버튼 구현**

기존 settings 로그아웃 로직 이식.

- [ ] **Step 6: AdminLayout에 MyPageModal 렌더링 추가**

```tsx
import { MyPageModal } from '@/features/auth/ui/MyPageModal'

// AdminLayout return문 내:
<>
  <div className="flex h-screen ...">
    {/* 기존 layout */}
  </div>
  <MyPageModal />
</>
```

- [ ] **Step 7: router에서 /settings 라우트 제거 및 리다이렉트 추가**

settings 라우트를 제거하고 `/settings` → `/` 리다이렉트 추가.

---

### Task 8: MyPage Backend API (요구사항 9)

**Files:**
- Modify: `01.backend/workspace/32_application/.../user/controller/UserController.java`
- Modify: `01.backend/workspace/32_application/.../user/v1/service/UserServiceV1.java`
- Modify: `01.backend/workspace/32_application/.../user/v1/dao/UserDaoV1.java`
- Create: `01.backend/workspace/32_application/.../resources/mapper/user/v1/UserMapperModV1.xml`
- Create: DTO 클래스 2개

- [ ] **Step 1: DTO 클래스 생성**

`UpdateMyInfoRequestDtoV1.java`:
```java
@Data
public class UpdateMyInfoRequestDtoV1 {
    @NotBlank
    private String userName;
}
```

`ChangePasswordRequestDtoV1.java`:
```java
@Data
public class ChangePasswordRequestDtoV1 {
    @NotBlank
    private String currentPassword;
    @NotBlank
    @Size(min = 8, max = 100)
    private String newPassword;
}
```

- [ ] **Step 2: UserDaoV1에 메서드 추가**

```java
void updateUserName(@Param("userNo") Long userNo, @Param("userName") String userName);
void updatePassword(@Param("userNo") Long userNo, @Param("password") String password);
String getPassword(@Param("userNo") Long userNo);
```

- [ ] **Step 3: UserMapperModV1.xml 생성/수정**

```xml
<update id="updateUserName">
    UPDATE scm_mmhr.tb_user
    SET user_name = #{userName}, mod_date = NOW(), mod_id = #{userNo}
    WHERE user_no = #{userNo} AND deleted_yn = 'N'
</update>

<update id="updatePassword">
    UPDATE scm_mmhr.tb_user
    SET password = #{password}, mod_date = NOW(), mod_id = #{userNo}
    WHERE user_no = #{userNo} AND deleted_yn = 'N'
</update>

<select id="getPassword" resultType="String">
    SELECT password FROM scm_mmhr.tb_user
    WHERE user_no = #{userNo} AND deleted_yn = 'N'
</select>
```

- [ ] **Step 4: UserServiceV1에 메서드 구현**

```java
public void updateMyInfo(Long userNo, UpdateMyInfoRequestDtoV1 dto) {
    userDaoV1.updateUserName(userNo, dto.getUserName());
}

public void changePassword(Long userNo, ChangePasswordRequestDtoV1 dto) {
    String stored = userDaoV1.getPassword(userNo);
    if (!passwordEncoder.matches(dto.getCurrentPassword(), stored)) {
        throw new BizException(CommonErrorCode.INVALID_PASSWORD);
    }
    String encoded = passwordEncoder.encode(dto.getNewPassword());
    userDaoV1.updatePassword(userNo, encoded);
}
```

- [ ] **Step 5: UserController에 엔드포인트 추가**

```java
@PostMapping("/updateMyInfo")
public ServerResponse<Void> updateMyInfo(@RequestBody @Valid UpdateMyInfoRequestDtoV1 dto) {
    Long userNo = RequestContextHolder.get().getUserNo();
    userServiceV1.updateMyInfo(userNo, dto);
    return ServerResponse.ok();
}

@PostMapping("/changePassword")
public ServerResponse<Void> changePassword(@RequestBody @Valid ChangePasswordRequestDtoV1 dto) {
    Long userNo = RequestContextHolder.get().getUserNo();
    userServiceV1.changePassword(userNo, dto);
    return ServerResponse.ok();
}
```

- [ ] **Step 6: Frontend API 함수 추가**

`entities/auth/api/` 또는 hs-common에 API 함수 추가:

```ts
async updateMyInfo(data: { userName: string }): Promise<void> {
  await client.post(buildUrl('/user/updateMyInfo'), data)
}

async changePassword(data: { currentPassword: string; newPassword: string }): Promise<void> {
  await client.post(buildUrl('/user/changePassword'), data)
}
```

---

### Task 9: RecordForm UI 개선 + 주최자 선택 (요구사항 6, 7)

**Files:**
- Modify: `21_hs-common/src/types/record.ts`
- Modify: `21_hs-common/src/types/ceremony.ts`
- Modify: `31_webadmin/src/features/record/ui/RecordForm.tsx`
- Modify: `31_webadmin/src/features/ceremony/ui/CeremonyForm.tsx`

- [ ] **Step 1: hs-common record.ts에 hostTypeCode 추가**

```ts
export interface MemonRecord {
  // ... 기존 필드
  hostTypeCode?: string  // 추가
}

export interface CreateRecordRequest {
  // ... 기존 필드
  hostTypeCode?: string  // 추가
}

export const HOST_TYPE_OPTIONS = [
  { value: 'SELF', label: '본인' },
  { value: 'OTHER', label: '타인' },
] as const

export const HOST_TYPE_MAP: Record<string, string> = {
  SELF: '본인',
  OTHER: '타인',
}
```

- [ ] **Step 2: hs-common ceremony.ts에 hostTypeCode 확인/추가**

Ceremony, CreateCeremonyRequest에 `hostTypeCode` 필드가 없으면 추가.

- [ ] **Step 3: RecordForm.tsx를 RelationshipForm 스타일로 리팩토링**

- 모든 필드를 `Field` + `FieldLabel` + `FieldError` 래핑
- `space-y-20` 간격 적용
- 필드 그룹핑: 기본정보 (유형/방향/주최자), 상세정보 (날짜/금액/메모)
- `<Input type="number">` → `<Controller>` + `<InputMoney>` 교체
- hostTypeCode SelectBox 필드 추가

Zod 스키마 업데이트:
```ts
const schema = z.object({
  recordTypeCode: z.string().min(1, '기록 유형을 선택해주세요'),
  directionCode: z.string().min(1, '방향을 선택해주세요'),
  hostTypeCode: z.string().optional(),
  recordDate: z.string().min(1, '날짜를 입력해주세요'),
  amount: z.string().optional(),
  memo: z.string().max(500).optional(),
})
```

금액 필드 (Controller 패턴):
```tsx
<Controller
  name="amount"
  control={form.control}
  render={({ field }) => (
    <InputMoney
      value={field.value ?? ''}
      onChange={field.onChange}
      step={10000}
      placeholder="금액을 입력하세요"
    />
  )}
/>
```

- [ ] **Step 4: CeremonyForm.tsx에 hostTypeCode 필드 추가**

동일한 패턴으로 SelectBox 추가.

- [ ] **Step 5: 빌드 확인**

Run: `cd 31_webadmin && npm run type-check`

---

### Task 10: 대시보드 RangeCalendar 범위 제어 (요구사항 12)

**Files:**
- Modify: `31_webadmin/src/shared/ui/period-filter.tsx`
- Modify: `31_webadmin/src/widgets/dashboard/DashboardStatBar.tsx`

- [ ] **Step 1: PeriodFilter에 DateRangePicker 연동**

```tsx
import { DateRangePicker } from '@/shared/ui/date-range-picker'

// PeriodFilter props에 추가:
interface PeriodFilterProps {
  // ... 기존 props
  onCustomDateChange?: (range: { startDate: string; endDate: string }) => void
}

// 날짜 영역 클릭 시 DateRangePicker 팝업:
const [showDatePicker, setShowDatePicker] = useState(false)

// 날짜 표시 영역에 onClick 연결:
<div onClick={() => setShowDatePicker(true)} className="cursor-pointer">
  <Calendar className="h-16 w-16 text-gray-3" />
  <span>{startDate}</span> — <span>{endDate}</span>
</div>

// DateRangePicker Popover:
{showDatePicker && (
  <DateRangePicker
    value={dateRange}
    onChange={(range) => {
      onCustomDateChange?.(range)
      setSelectedPreset('custom')
      setShowDatePicker(false)
    }}
  />
)}
```

- [ ] **Step 2: 프리셋에 '사용자 지정' 옵션 추가**

```ts
const PRESET_OPTIONS = [
  { value: 'all', label: '전체' },
  { value: 'this-month', label: '이번 달' },
  { value: 'last-month', label: '지난 달' },
  { value: 'this-year', label: '올해' },
  { value: 'last-year', label: '작년' },
  { value: 'custom', label: '사용자 지정' },
]
```

- [ ] **Step 3: DashboardStatBar에서 커스텀 날짜 범위를 데이터 조회에 반영**

커스텀 날짜 선택 시 `useHomeSummary(params)` 호출에 startDate/endDate 전달.

---

## Phase 4: 관계연결 기능 (요구사항 10)

### Task 11: Database DDL 작성

**Files:**
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/tb_relationship_link.sql`

- [ ] **Step 1: DDL 파일 생성**

설계 문서의 `tb_relationship_link` DDL을 그대로 작성.

---

### Task 12: Backend — VO, DTO, DAO 생성

**Files:**
- Create: `32_application/.../vo/mmhr/RelationshipLinkVO.java`
- Create: `32_application/.../relationshipLink/v1/dto/` (6+ DTO 클래스)
- Create: `32_application/.../relationshipLink/v1/dao/RelationshipLinkDaoV1.java`
- Create: `32_application/.../resources/mapper/relationshipLink/v1/RelationshipLinkMapperV1.xml`
- Create: `32_application/.../resources/mapper/relationshipLink/v1/RelationshipLinkMapperModV1.xml`

- [ ] **Step 1: RelationshipLinkVO 생성**

```java
@Data @EqualsAndHashCode(callSuper = true)
public class RelationshipLinkVO extends BaseVO {
    private Long linkNo;
    private Long requesterUserNo;
    private Long responderUserNo;
    private String responderEmail;
    private String inviteCode;
    private String statusCode;
    private LocalDateTime expireDate;
    private LocalDateTime acceptedDate;
    private String deletedYn;
}
```

- [ ] **Step 2: DTO 클래스 생성**

- `SearchUserRequestDtoV1` { email }
- `SearchUserResponseDtoV1` { userNo, userName, maskedEmail }
- `CreateLinkRequestDtoV1` { responderEmail }
- `CreateLinkResponseDtoV1` { linkNo, inviteCode, expireDate }
- `AcceptLinkRequestDtoV1` { inviteCode }
- `LinkItemResponseDtoV1` { linkNo, userName, email, statusCode, regDate, expireDate }
- `ActiveLinkResponseDtoV1` { linkNo, userName, email, acceptedDate }

- [ ] **Step 3: DAO 인터페이스 생성**

```java
@Mapper
public interface RelationshipLinkDaoV1 {
    void insertLink(RelationshipLinkVO vo);
    RelationshipLinkVO selectByInviteCode(@Param("inviteCode") String inviteCode);
    void updateStatus(@Param("linkNo") Long linkNo, @Param("statusCode") String statusCode,
                      @Param("responderUserNo") Long responderUserNo);
    List<LinkItemResponseDtoV1> selectSentList(@Param("userNo") Long userNo, @Param("pageNumber") int pageNumber, @Param("pageSize") int pageSize);
    int countSentList(@Param("userNo") Long userNo);
    List<LinkItemResponseDtoV1> selectReceivedList(@Param("userNo") Long userNo, @Param("pageNumber") int pageNumber, @Param("pageSize") int pageSize);
    int countReceivedList(@Param("userNo") Long userNo);
    List<ActiveLinkResponseDtoV1> selectActiveLinks(@Param("userNo") Long userNo);
    RelationshipLinkVO selectByLinkNo(@Param("linkNo") Long linkNo);
    int checkDuplicateLink(@Param("requesterUserNo") Long requesterUserNo, @Param("responderEmail") String email);
    Long selectUserNoByEmail(@Param("email") String email);
    String selectUserNameByNo(@Param("userNo") Long userNo);
}
```

- [ ] **Step 4: MyBatis Mapper XML 생성**

SELECT용 `RelationshipLinkMapperV1.xml` 및 INSERT/UPDATE용 `RelationshipLinkMapperModV1.xml` 작성.

---

### Task 13: Backend — Service + Controller

**Files:**
- Create: `32_application/.../relationshipLink/v1/service/RelationshipLinkServiceV1.java`
- Create: `32_application/.../relationshipLink/controller/RelationshipLinkController.java`

- [ ] **Step 1: Service 구현**

핵심 비즈니스 로직:
- `createLink`: 이메일로 사용자 검색 → 중복 확인 → UUID 초대코드 생성 → DB 저장 → MQ 이메일 발송
- `acceptLink`: 초대코드 검증 → 만료 확인 → status ACCEPTED 업데이트
- `rejectLink`: status REJECTED 업데이트
- `getLinked*`: ACCEPTED 상태 확인 → responder_user_no 기반 데이터 조회

- [ ] **Step 2: Controller 구현**

설계 문서의 14개 API 엔드포인트를 Controller에 구현. 기존 프로젝트 패턴(ServiceResolver, version) 따름.

- [ ] **Step 3: MQ 리소스 설정 (이메일 발송용)**

- `MqConstants.java`에 상수 추가
- `MqResourceConfig.java`에 Bean 정의
- `rabbitmq-definitions.json` 업데이트

---

### Task 14: Frontend — hs-common 타입 + API

**Files:**
- Create: `21_hs-common/src/types/relationshipLink.ts`
- Create: `21_hs-common/src/api/entities/relationshipLinkApi.ts`
- Modify: `21_hs-common/src/types/index.ts` (export 추가)
- Modify: `21_hs-common/src/api/entities/index.ts` (export 추가)

- [ ] **Step 1: relationshipLink.ts 타입 정의**

```ts
export interface LinkItem {
  linkNo: number
  userName: string
  email: string
  statusCode: string
  regDate: string
  expireDate: string
}

export interface ActiveLink {
  linkNo: number
  userName: string
  email: string
  acceptedDate: string
}

export interface SearchUserResult {
  userNo: number
  userName: string
  maskedEmail: string
}

export interface CreateLinkResponse {
  linkNo: number
  inviteCode: string
  expireDate: string
}

// Request types
export interface CreateLinkRequest { responderEmail: string }
export interface AcceptLinkRequest { inviteCode: string }
export interface LinkedDataRequest { linkNo: number; pageNumber?: number; pageSize?: number }

// Status constants
export const LINK_STATUS_MAP: Record<string, string> = {
  PENDING: '대기',
  ACCEPTED: '수락',
  REJECTED: '거절',
  EXPIRED: '만료',
  CANCELLED: '취소',
}
```

- [ ] **Step 2: relationshipLinkApi.ts API 클라이언트 생성**

기존 API factory 패턴을 따라 구현.

---

### Task 15: Frontend — 관계연결 페이지

**Files:**
- Create: `31_webadmin/src/entities/relationshipLink/model/types.ts`
- Create: `31_webadmin/src/entities/relationshipLink/model/queries.ts`
- Create: `31_webadmin/src/entities/relationshipLink/api/relationshipLinkApi.ts`
- Create: `31_webadmin/src/features/relationshipLink/ui/LinkRequestTab.tsx`
- Create: `31_webadmin/src/features/relationshipLink/ui/ReceivedRequestsTab.tsx`
- Create: `31_webadmin/src/features/relationshipLink/ui/ActiveLinksTab.tsx`
- Create: `31_webadmin/src/features/relationshipLink/ui/LinkedDataDialog.tsx`
- Create: `31_webadmin/src/app/(authenticated)/relationship-link/page.tsx`
- Modify: `31_webadmin/src/shared/config/menu-config.ts` (메뉴 추가)
- Modify: `31_webadmin/src/app/router/` (라우트 추가)
- Modify: `31_webadmin/src/shared/components/data-grid.tsx` (readOnly prop)

- [ ] **Step 1: entities 레이어 생성**

types 재수출, API 인스턴스 생성, TanStack Query hooks 정의.

- [ ] **Step 2: DataGrid에 readOnly prop 추가**

```ts
export interface DataGridProps<...> {
  // ... 기존
  readOnly?: boolean  // 추가
}

// readOnly가 true이면 selectable을 강제 false로 처리
const selectableConfig = useMemo(() => {
  if (readOnly) return undefined
  // ... 기존 로직
}, [selectable, readOnly])
```

- [ ] **Step 3: LinkRequestTab 구현**

이메일 검색 Input + 검색 결과 표시 + 연결 요청 버튼.

- [ ] **Step 4: ReceivedRequestsTab 구현**

받은 요청 카드 목록 + 수락/거절 버튼 + 만료 타이머 표시.

- [ ] **Step 5: ActiveLinksTab 구현**

활성 연결 카드 목록 + 기록 보기 버튼 + 연결 해제 버튼.

- [ ] **Step 6: LinkedDataDialog 구현**

탭 (관계 | 기록 | 경조사 | 일정) + 각 탭에 readOnly DataGrid.

- [ ] **Step 7: 메인 페이지 + 라우트 + 메뉴 연결**

관계연결 페이지에 3개 탭 구성. menu-config에 Link 아이콘 메뉴 추가. router에 라우트 추가.

- [ ] **Step 8: 빌드 확인**

Run: `cd 31_webadmin && npm run type-check && npm run lint`

---

## 검증

- [ ] **전체 빌드**: `cd 31_webadmin && npm run build`
- [ ] **Backend 빌드**: `cd 32_application && ./gradlew build`
- [ ] **hs-common 변경 영향**: webadmin + appuser 양쪽 타입체크
