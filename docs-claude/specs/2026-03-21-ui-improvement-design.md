# MEMON UI/기능 개선 설계 문서

**작성일:** 2026-03-21
**범위:** Frontend (31_webadmin) + Backend (32_application) + Database (11_database)

---

## 1. 요구사항 요약

| # | 요구사항 | 카테고리 | 변경 범위 |
|---|---------|---------|----------|
| 1 | Grid 컬럼 너비 조정 + 최소너비 | UI 개선 | Frontend |
| 2 | 로딩 progress 크기 2배 | UI 개선 | Frontend |
| 3 | Modal title 글자색 수정 | UI 버그 | Frontend |
| 4 | 금액 input 상하버튼 위치 고정 | UI 버그 | Frontend |
| 5 | 금액 증감 기본값 10000 | UI 개선 | Frontend |
| 6 | 경조사 기록 시 주최자(본인/타인) 선택 | 기능 추가 | Frontend |
| 7 | 새기록추가 UI → 관계관리와 동일 | UI 개선 | Frontend |
| 8 | "김" → myPage 버튼, 설정 메뉴 제거 | UI 구조 변경 | Frontend |
| 9 | myPage 이름/비밀번호 변경 | 기능 추가 | Frontend + Backend |
| 10 | 관계연결 기능 | 신규 기능 | Frontend + Backend + Database |
| 11 | 로딩 UI 통일 | UI 개선 | Frontend |
| 12 | 대시보드 RangeCalendar 범위 제어 | UI 개선 | Frontend |

---

## 2. Category A: CSS/UI 수정 (Frontend Only)

### 2.1 Grid 컬럼 너비 조정 + 최소너비 (요구사항 1)

**현재 상태:**
- `DataGrid` (shared/components/data-grid.tsx): TanStack Table 기반, `table-fixed` 사용
- 컬럼 너비는 `DataGridColumn.width`로 고정, 리사이즈 불가
- 헤더 텍스트가 컬럼 너비보다 길면 2줄로 떨어짐

**변경 사항:**
1. TanStack Table의 `enableColumnResizing` + `columnResizeMode: 'onChange'` 활성화
2. `table-fixed` → 리사이즈 모드에서는 제거 (TanStack column resizing은 explicit width 관리 필요)
3. 헤더에 리사이즈 핸들(드래그 가능한 세로선) 추가 — CSS cursor: `col-resize`, hover 시 시각적 피드백
4. `DataGridColumn`에 `minWidth` prop 추가
5. 컬럼 타입별 기본 최소너비:
   - checkbox: 50px (고정)
   - 날짜 컬럼: 120px
   - 금액 컬럼: 100px
   - 이름 컬럼: 80px
   - 메모/텍스트: 120px
   - 기본값: 80px
6. 헤더 `whitespace-nowrap`은 이미 적용됨 — 최소너비로 2줄 문제 해결

**영향 파일:**
- `31_webadmin/src/shared/components/data-grid.tsx`

---

### 2.2 로딩 progress 크기 2배 (요구사항 2)

**현재 상태:**
- `LoadingSpinner` sizeMap: sm=h-4 w-4, md=h-8 w-8, lg=h-12 w-12

**변경 사항:**
- sizeMap 변경:
  - sm: `h-8 w-8`
  - md: `h-16 w-16`
  - lg: `h-24 w-24`

**주의:** 기존 LoadingSpinner 사용처 전체에 영향. 기존 인라인 스피너(sm/md)가 과도하게 커질 수 있으므로, 변경 후 전체 사용처를 확인하여 적절한 size prop으로 조정.

**영향 파일:**
- `31_webadmin/src/shared/ui/loading-spinner.tsx`

---

### 2.3 Modal title 글자색 수정 (요구사항 3)

**현재 상태:**
- DialogHeader: `bg-brand-deep` 배경
- DialogTitle: `text-white` 글자색
- **문제:** `--brand-deep` CSS 변수가 정의되지 않아 배경이 투명/흰색 → 흰글자가 보이지 않음
- `bg-brand-deep`는 dialog 외에 intro 위젯들(HeroSection, ValuePropositionSection, PublicStatsSection)에서도 `text-brand-deep`으로 사용됨

**변경 사항:**
1. `index.css`에 `--brand-deep` CSS 변수 추가 (테마별):
   - 기본(lemon-gold): `#92400E` (앰버 다크)
   - deep-blue: `#1E3A5F`
   - emerald: `#064E3B`
   - monochrome: `#111827`
2. **hs-common preset**(`21_hs-common/src/tailwind/preset.js`)에 `'brand-deep': 'var(--brand-deep)'` 색상 추가
   - hs-common 변경이므로 **webadmin + appuser 양쪽 빌드 확인 필수**

**영향 파일:**
- `31_webadmin/src/index.css`
- `21_hs-common/src/tailwind/preset.js`

---

### 2.4 + 2.5 금액 input 스테퍼 (요구사항 4, 5)

**현재 상태:**
- RecordForm에서 `<Input type="number" {...form.register('amount', { valueAsNumber: true })}>`를 사용
- 브라우저 기본 spin button이 값 입력 시 위치 틀어짐
- `InputMoney`는 `type="text"` + `useNumberInput(type: 'money')` 사용, 콤마 포맷팅 지원

**변경 사항:**
1. 브라우저 기본 spin button 숨김 (CSS: `input::-webkit-outer-spin-button` 등)
2. `InputMoney`에 커스텀 스테퍼 UI 추가:
   - `stepper` prop (default: true) — 스테퍼 표시 여부
   - `step` prop (default: 10000) — 증감 단위
   - input 우측에 ▲/▼ 버튼 absolute 배치, 값 변경과 무관하게 위치 고정
3. 스테퍼 로직 구현:
   - `parseCurrency(displayValue)` → 숫자값 → `+/- step` → `formatCurrency()` → `onChange()` 호출
   - `useNumberInput`에 `increment(step)` / `decrement(step)` 함수 추가 또는 InputMoney 내에서 직접 처리
   - min/max 범위 존중
4. **RecordForm 변경:**
   - `<Input type="number" {...register('amount', { valueAsNumber: true })}>` → `<Controller>` + `<InputMoney>` 교체
   - React Hook Form `register` → `Controller`의 `field.value`/`field.onChange` 패턴 사용
   - `Controller`의 `onChange`에서 string → number 변환 처리

**영향 파일:**
- `31_webadmin/src/shared/ui/input-money.tsx`
- `31_webadmin/src/shared/hooks/useNumberInput.ts` (increment/decrement 함수 추가)
- `31_webadmin/src/shared/lib/input-number.ts` (parseCurrency 활용)
- `31_webadmin/src/features/record/ui/RecordForm.tsx`
- `31_webadmin/src/index.css` (spin button 숨김 CSS)

---

### 2.6 로딩 UI 통일 (요구사항 11)

**현재 상태:**
| 페이지 | 로딩 표시 |
|--------|----------|
| 관계관리 | `<LoadingSpinner />` (스피너만) |
| 대시보드 | `"불러오는 중..."` (문구만) |
| 기록관리 | `"불러오는 중..."` (문구만) |
| 경조사관리 | `"불러오는 중..."` (문구만) |
| DataGrid | `"로딩 중..."` (문구만) |

**변경 사항:**
- 모든 로딩 상태를 `<LoadingSpinner size="lg" message="데이터를 불러오는 중..." />`로 통일
- DataGrid 내부 isLoading 상태도 LoadingSpinner 사용으로 변경

**영향 파일:**
- `31_webadmin/src/app/(authenticated)/page.tsx` (대시보드)
- `31_webadmin/src/app/(authenticated)/records/page.tsx`
- `31_webadmin/src/app/(authenticated)/ceremonies/page.tsx`
- `31_webadmin/src/app/(authenticated)/relations/page.tsx` (message 추가)
- `31_webadmin/src/shared/components/data-grid.tsx`
- `31_webadmin/src/widgets/dashboard/UpcomingSection.tsx`
- `31_webadmin/src/widgets/dashboard/RecentRecordsSection.tsx`

---

## 3. Category B: Frontend UI 구조 변경

### 3.1 새기록추가 UI → 관계관리와 동일 (요구사항 7)

**현재 상태:**
- RecordForm: 간소한 Dialog — Field 래핑이 일부 누락, 간격 불균일
- RelationshipForm: 풍부한 Dialog — 모든 필드 `Field` 래핑, 일관된 간격(space-y-20), 섹션 구분

**구체적 변경:**
1. 모든 필드를 `Field` + `FieldLabel` + `FieldError` 패턴으로 래핑
2. `space-y-20` 간격 적용 (RelationshipForm과 동일)
3. 관련 필드 그룹핑:
   - 기본정보 섹션: 기록유형, 방향, 주최자(hostTypeCode)
   - 상세정보 섹션: 날짜, 금액, 메모
4. `InputMoney` + `Controller` 패턴으로 금액 필드 교체 (요구사항 4/5와 연동)
5. 기존 기능 유지 (create/edit 모드, validation)

**영향 파일:**
- `31_webadmin/src/features/record/ui/RecordForm.tsx`

---

### 3.2 myPage 버튼 + 설정 메뉴 → myPage modal (요구사항 8)

**현재 상태:**
- Header: 사용자 아바타(성 첫글자, 예: "김") 표시
- Sidebar: Settings 메뉴 항목 존재 → /settings 페이지
- Settings 페이지: 내 정보(조회), 계정연동(placeholder), 알림설정(placeholder), 테마, 로그아웃

**변경 사항:**
1. Header의 아바타 → "마이페이지" 텍스트 또는 User 아이콘 버튼으로 변경
2. 클릭 시 MyPage Dialog(modal) 열림
3. Sidebar에서 "설정" 메뉴 항목 제거
4. /settings 라우트 제거 + `/settings` → `/` 리다이렉트 추가 (북마크 대응)
5. MyPage modal 내용:
   - 내 정보 (이름 수정 가능)
   - 비밀번호 변경
   - 테마 변경
   - 로그아웃

**FSD 레이어 설계:**
- `MyPageModal` 컴포넌트 위치: `features/auth/ui/MyPageModal.tsx`
- Header(widget)에서 직접 import하면 FSD 위반 (widgets → features 금지)
- **해결:** `AuthenticatedLayout`(app 레이어)에서 MyPageModal을 렌더링
  - Zustand store (`shared/stores/myPageStore.ts`)로 modal open 상태 관리
  - Header는 store의 `open()` 함수만 호출
  - Layout은 store를 구독하여 MyPageModal 렌더링

**영향 파일:**
- `31_webadmin/src/widgets/header/Header.tsx`
- `31_webadmin/src/shared/config/menu-config.ts`
- `31_webadmin/src/shared/stores/myPageStore.ts` (신규)
- `31_webadmin/src/features/auth/ui/MyPageModal.tsx` (신규)
- `31_webadmin/src/app/layouts/AuthenticatedLayout.tsx` (또는 AdminLayout)
- `31_webadmin/src/app/router/` (settings 라우트 제거 + 리다이렉트)

---

### 3.3 대시보드 RangeCalendar 범위 제어 (요구사항 12)

**현재 상태:**
- PeriodFilter: 프리셋 드롭다운 + 날짜 표시 (날짜 클릭 핸들러 미구현: `onClick={() => {/* Date picker will be wired later */}}`)
- DateRangePicker 컴포넌트가 이미 존재

**변경 사항:**
1. PeriodFilter의 날짜 영역 클릭 시 DateRangePicker 팝업 열림
2. 커스텀 날짜 선택 시 프리셋 드롭다운은 "사용자 지정"으로 변경
3. 선택된 날짜 범위가 대시보드 데이터 조회에 반영

**영향 파일:**
- `31_webadmin/src/shared/ui/period-filter.tsx`
- `31_webadmin/src/widgets/dashboard/DashboardStatBar.tsx`

---

## 4. Category C: Frontend + Backend 변경

### 4.1 경조사 기록 시 주최자 선택 (요구사항 6)

**현재 상태:**
- Database: `tb_record.host_type_code` 컬럼 존재 (SELF/OTHER)
- Backend: `RecordVO.hostTypeCode` 필드 존재, DTO에 포함
- Frontend:
  - `MemonRecord` interface에 `hostTypeCode` 필드 **없음**
  - `CreateRecordRequest` / `UpdateRecordRequest`에 `hostTypeCode` 필드 **없음**
  - RecordForm에 hostTypeCode 필드 미노출
  - CeremonyForm의 hostTypeCode 필드 상태도 확인 필요

**변경 사항:**

1. **hs-common 타입 수정** (`21_hs-common/src/types/record.ts`):
   - `MemonRecord`에 `hostTypeCode?: string` 필드 추가
   - `RecordDetail`은 `MemonRecord` 확장이므로 자동 포함
   - `CreateRecordRequest`에 `hostTypeCode?: string` 추가
   - `UpdateRecordRequest`는 `CreateRecordRequest` 확장이므로 자동 포함
   - `HOST_TYPE_OPTIONS` 상수 추가: `[{ value: 'SELF', label: '본인' }, { value: 'OTHER', label: '타인' }]`
   - `HOST_TYPE_MAP` 상수 추가: `{ SELF: '본인', OTHER: '타인' }`

2. **hs-common ceremony 타입 확인** (`21_hs-common/src/types/ceremony.ts`):
   - `Ceremony`, `CreateCeremonyRequest` 등에 `hostTypeCode` 필드 확인/추가

3. **RecordForm 변경** (`31_webadmin/src/features/record/ui/RecordForm.tsx`):
   - Zod 스키마에 `hostTypeCode: z.string().optional()` 추가
   - SelectBox 필드 추가 (HOST_TYPE_OPTIONS 사용)
   - `handleFormSubmit`에 `hostTypeCode` 포함

4. **CeremonyForm 변경** (필요시)

**영향 파일:**
- `21_hs-common/src/types/record.ts`
- `21_hs-common/src/types/ceremony.ts` (확인/수정)
- `31_webadmin/src/features/record/ui/RecordForm.tsx`
- `31_webadmin/src/features/ceremony/ui/CeremonyForm.tsx` (확인/수정)

---

### 4.2 myPage 이름/비밀번호 변경 API (요구사항 9)

**현재 상태:**
- Backend: `UserController.getMyInfo()` 조회만 존재, 수정 API 없음

**변경 사항:**

**Backend:**
1. `UserController`에 API 추가:
   - `POST /user/updateMyInfo` (buildApiUrl이 `/api/v1/` prefix를 자동 추가)
     - Request: `{ userName: string }`
     - Response: `ServerResponse<void>`
   - `POST /user/changePassword`
     - Request: `{ currentPassword: string, newPassword: string }`
     - Response: `ServerResponse<void>`
2. `UserServiceV1` 구현:
   - `updateMyInfo`: user_no 기반 이름 업데이트
   - `changePassword`: 현재 비밀번호 검증 → 새 비밀번호 해시 후 업데이트
3. `UserDaoV1` + Mapper XML 추가

**Frontend:**
1. MyPageModal 내 이름 수정 폼
2. 비밀번호 변경 폼 (현재 비밀번호 + 새 비밀번호 + 확인)
3. Zod 검증 (비밀번호 일치, 최소 길이 등)

**영향 파일:**
- Backend:
  - `32_application/.../user/controller/UserController.java`
  - `32_application/.../user/v1/service/UserServiceV1.java`
  - `32_application/.../user/v1/dao/UserDaoV1.java`
  - `32_application/.../resources/mapper/user/v1/UserMapperModV1.xml`
  - DTO 클래스 신규 생성 (UpdateMyInfoRequestDtoV1, ChangePasswordRequestDtoV1)
- Frontend:
  - `31_webadmin/src/features/auth/ui/MyPageModal.tsx` (신규)
  - `31_webadmin/src/entities/auth/api/` (API 함수 추가)
  - `21_hs-common/src/types/auth.ts` (타입 추가)

---

## 5. Category D: 신규 기능 — 관계연결 (요구사항 10)

### 5.1 개요

사용자가 다른 사용자를 이메일로 검색하여 관계 연결을 요청하고, 상대방이 초대코드를 승인하면 발신자가 수신자의 기록 정보를 읽기 전용으로 조회할 수 있는 기능.

### 5.2 비즈니스 규칙

- 발신자가 이메일로 상대방을 검색하여 연결 요청
- 초대코드가 생성되어 상대방 이메일로 발송
- 초대코드 만료 시간: **2시간**
- 수신자가 초대코드를 승인하면 연결 성립
- 연결 성립 후 발신자는 수신자의 데이터를 **읽기 전용**으로 조회 가능:
  - 관계 목록
  - 기록 목록
  - 경조사 목록
  - 일정 목록
- 공유 데이터는 **조회만 가능** (수정/삭제 불가)
- 양방향 공유가 아닌 **단방향**: 발신자 → 수신자 데이터 조회
- 상호 공유를 원하면 수신자도 별도로 연결 요청해야 함

### 5.3 Database 설계

```sql
-- scm_mmhr.tb_relationship_link
CREATE TABLE scm_mmhr.tb_relationship_link (
    link_no             BIGSERIAL       PRIMARY KEY,
    requester_user_no   BIGINT          NOT NULL,       -- 요청자 (발신자)
    responder_user_no   BIGINT,                         -- 응답자 (수신자, 승인 전 NULL 가능)
    responder_email     VARCHAR(200)    NOT NULL,        -- 수신자 이메일
    invite_code         VARCHAR(32)     UNIQUE NOT NULL, -- 초대코드 (UNIQUE 제약이 인덱스 역할)
    status_code         VARCHAR(20)     NOT NULL DEFAULT 'PENDING',
                        -- PENDING: 대기 / ACCEPTED: 수락 / REJECTED: 거절 / EXPIRED: 만료 / CANCELLED: 취소
    expire_date         TIMESTAMP       NOT NULL,        -- 초대코드 만료 시간 (생성 + 2시간)
    accepted_date       TIMESTAMP,                       -- 수락 일시
    reg_date            TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    reg_id              VARCHAR(50),
    mod_date            TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    mod_id              VARCHAR(50),
    deleted_yn          CHAR(1)         DEFAULT 'N',
    CONSTRAINT fk_link_requester FOREIGN KEY (requester_user_no) REFERENCES scm_mmhr.tb_user(user_no),
    CONSTRAINT fk_link_responder FOREIGN KEY (responder_user_no) REFERENCES scm_mmhr.tb_user(user_no)
);

CREATE INDEX idx_link_requester ON scm_mmhr.tb_relationship_link(requester_user_no);
CREATE INDEX idx_link_responder ON scm_mmhr.tb_relationship_link(responder_user_no);
CREATE INDEX idx_link_status ON scm_mmhr.tb_relationship_link(status_code);
-- invite_code UNIQUE 제약이 자동으로 인덱스를 생성하므로 별도 인덱스 불필요
```

### 5.4 초대코드 만료 처리

**방식:** 이중 전략 (Reactive + Proactive)
1. **Reactive (필수):** `acceptLink` 호출 시 `expire_date < NOW()` 체크, 만료 시 거부 응답 반환
2. **Proactive (선택):** System Server에 배치 작업 추가 — 매시간 `PENDING` + `expire_date < NOW()` 레코드를 `EXPIRED`로 벌크 업데이트
   - 이는 목록 조회 시 만료된 항목이 여전히 PENDING으로 표시되는 것을 방지

### 5.5 이메일 발송

**방식:** Application Server → MQ → System Server → 이메일 발송

**MQ 리소스 (신규 추가 필수):**
- Exchange: `memon.notification` (기존 사용 가능하면 재활용)
- Queue: `memon.email.invite`
- Routing key: `email.invite`

**MQ 리소스 변경 시 필수 업데이트:**
- `MqConstants.java`: 상수 추가
- `MqResourceConfig.java`: Bean 정의
- `05_infra/INFRASTRUCTURE.md`: 문서 업데이트
- `rabbitmq-definitions.json`: 리소스 정의

**이메일 발송 실패 처리:**
- Link 레코드는 생성되지만 `status_code`는 `PENDING` 유지
- 이메일 발송 실패 시 사용자에게 "초대 링크가 생성되었으나 이메일 발송에 실패했습니다. 초대코드를 직접 전달해주세요." 안내
- 초대코드는 응답에 포함하여 사용자가 직접 공유 가능

### 5.6 Backend API 설계

**Controller:** `RelationshipLinkController`
**Base Path:** `/relationship-link` (buildApiUrl이 prefix 자동 추가)

| Method | Endpoint | 설명 | Request | Response |
|--------|---------|------|---------|----------|
| POST | `/searchUser` | 이메일로 사용자 검색 | `{ email: string }` | `{ userNo, userName, email(마스킹) }` |
| POST | `/createLink` | 연결 요청 (초대코드 생성 + 이메일 발송) | `{ responderEmail: string }` | `{ linkNo, inviteCode, expireDate }` |
| POST | `/acceptLink` | 초대 수락 | `{ inviteCode: string }` | `ServerResponse<void>` |
| POST | `/rejectLink` | 초대 거절 | `{ inviteCode: string }` | `ServerResponse<void>` |
| POST | `/cancelLink` | 연결 요청 취소 (발신자) | `{ linkNo: number }` | `ServerResponse<void>` |
| POST | `/getSentList` | 보낸 연결 요청 목록 | `{ ...pagination }` (body) | `PageDto<LinkItem>` |
| POST | `/getReceivedList` | 받은 연결 요청 목록 | `{ ...pagination }` (body) | `PageDto<LinkItem>` |
| POST | `/getActiveLinks` | 활성 연결 목록 | `{ ...pagination }` (body) | `PageDto<ActiveLink>` |
| POST | `/deleteLink` | 연결 해제 | `{ linkNo: number }` | `ServerResponse<void>` |
| POST | `/getLinkedRecords` | 연결된 사용자의 기록 조회 | `{ linkNo, ...pagination }` | `PageDto<MemonRecord>` |
| POST | `/getLinkedRelationships` | 연결된 사용자의 관계 조회 | `{ linkNo, ...pagination }` | `PageDto<Relationship>` |
| POST | `/getLinkedCeremonies` | 연결된 사용자의 경조사 조회 | `{ linkNo, ...pagination }` | `PageDto<Ceremony>` |
| POST | `/getLinkedSchedules` | 연결된 사용자의 일정 조회 | `{ linkNo, ...pagination }` | `PageDto<Schedule>` |

> **참고:** 기존 프로젝트 패턴이 조회 포함 대부분 POST 사용 (authApi, recordApi 등)이므로 POST로 통일.

**Service 비즈니스 로직:**
- `createLink`:
  - 이메일로 사용자 존재 확인
  - 중복 연결 요청 방지 (이미 PENDING/ACCEPTED 상태 존재 시 거부)
  - UUID 기반 초대코드 생성 (32자 hex)
  - 만료 시간 = 현재 + 2시간
  - MQ를 통해 이메일 발송 요청
- `acceptLink`:
  - 초대코드 유효성 확인
  - **만료 확인** (`expire_date < NOW()` → EXPIRED 처리 후 거부)
  - status → ACCEPTED, accepted_date 설정
  - responder_user_no 설정
- `getLinked*` 조회:
  - 연결 상태가 ACCEPTED인지 확인
  - requester_user_no가 현재 사용자인지 확인
  - responder_user_no 기반 데이터 조회 (읽기 전용)

### 5.7 Frontend 설계

**신규 페이지:** `/relationship-link` (관계연결)

**Sidebar 메뉴 추가:** "관계연결" (Link icon), 관계관리 아래 배치

**페이지 구성 (탭 3개):**

1. **연결 요청 탭**
   - 이메일 검색 input + 검색 버튼
   - 검색 결과 표시 (이름, 마스킹된 이메일)
   - "연결 요청" 버튼 → 확인 dialog → API 호출

2. **받은 요청 탭**
   - 받은 연결 요청 목록 (카드 형태)
   - 각 카드: 요청자 이름, 요청일, 만료까지 남은 시간
   - 수락/거절 버튼

3. **내 연결 탭**
   - 활성 연결 목록 (카드 형태)
   - 각 카드: 연결된 사용자 이름, 연결일
   - "기록 보기" 버튼 → 연결된 사용자의 데이터 조회 Dialog
   - "연결 해제" 버튼

**연결된 사용자 데이터 조회 Dialog:**
- 읽기 전용 데이터 뷰어
- 탭 구성: 관계 | 기록 | 경조사 | 일정
- DataGrid에 `readOnly` prop 추가: true일 때 checkbox/action 컬럼 숨김
- 수정/삭제/추가 버튼 렌더링하지 않음

**영향 파일:**
- Database:
  - `01.backend/workspace/11_database/tables/scm_mmhr/tb_relationship_link.sql` (신규)
- Backend:
  - `01.backend/workspace/32_application/.../relationshipLink/` (신규 모듈)
  - Controller, ServiceResolver, ServiceV1, DaoV1, Mapper XML, VO, DTO
  - MQ 관련: MqConstants, MqResourceConfig, rabbitmq-definitions.json
  - `01.backend/workspace/33_system/` (이메일 발송 Consumer)
- Frontend:
  - `21_hs-common/src/types/relationshipLink.ts` (신규)
  - `21_hs-common/src/api/entities/relationshipLinkApi.ts` (신규)
  - `31_webadmin/src/entities/relationshipLink/` (신규)
  - `31_webadmin/src/features/relationshipLink/` (신규)
  - `31_webadmin/src/app/(authenticated)/relationship-link/page.tsx` (신규)
  - `31_webadmin/src/shared/config/menu-config.ts` (메뉴 추가)
  - `31_webadmin/src/shared/components/data-grid.tsx` (readOnly prop 추가)
  - `31_webadmin/src/app/router/` (라우트 추가)

---

## 6. 구현 순서 (권장)

### Phase 1: UI 버그 수정 + CSS 개선 (독립, 병렬 가능)
- 요구사항 2: 로딩 크기 2배
- 요구사항 3: Modal title 글자색
- 요구사항 11: 로딩 UI 통일

### Phase 2: Input/Grid 개선 (순차)
- 요구사항 4 + 5: 금액 input 스테퍼
- 요구사항 1: Grid 컬럼 리사이즈

### Phase 3: UI 구조 변경 (순차)
- 요구사항 8: myPage 버튼/modal (store + layout + modal)
- 요구사항 9: myPage 이름/비밀번호 변경 (backend API + frontend)
- 요구사항 7: 새기록추가 UI 변경
- 요구사항 6: 주최자 선택 추가
- 요구사항 12: RangeCalendar 범위 제어

### Phase 4: 신규 기능 (순차)
- 요구사항 10: 관계연결 (Database → Backend → Frontend)

---

## 7. 제약 사항

- `00.database/` 디렉토리는 작업 대상에서 제외 → DB 스크립트는 `01.backend/workspace/11_database/`에 작성
- commit & push는 별도 명령 시에만 수행
- hs-common 변경 시 webadmin + appuser 빌드 확인 필수
- FSD 레이어 의존성 방향 준수: app → widgets → features → entities → shared
- MyPageModal은 app 레이어에서 렌더링, widget(Header)은 store 함수만 호출
- 기존 프로젝트 API 패턴: 조회 포함 POST 사용 (body로 파라미터 전달)
- MQ 리소스 변경 시 4개 파일 동시 업데이트 필수
