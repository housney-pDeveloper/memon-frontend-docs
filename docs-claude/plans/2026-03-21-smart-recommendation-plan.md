# MEMON 스마트 금액 추천 엔진 구현 계획

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 3단계 금액 추천 파이프라인, 물가 환산 계산기, 예산 플래너, 스마트 알림, 관계 타임라인 등 경조사 재무 어드바이저 기능을 구현한다.

**Architecture:** 개인 데이터 + 집단 통계(Spring Scheduler 매일 집계) + 한국 정서 라운딩. CPI/이자율 테이블 기반 물가 환산. FSD 아키텍처 준수.

**Tech Stack:** React 19, TypeScript, Recharts, Zustand, React Hook Form + Zod, Tailwind CSS, Spring Boot 3.5, MyBatis, PostgreSQL, Spring Scheduler

**Spec:** `01_docs/docs-claude/specs/2026-03-21-smart-recommendation-design.md`

---

## Phase A: 기반 인프라 (Database + Backend 기초)

### Task 1: Database DDL 생성

**Files:**
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/tb_cpi_index.sql`
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/tb_statistics_aggregate.sql`
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/tb_recommendation_default.sql`
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/tb_budget.sql`
- Create: `01.backend/workspace/11_database/tables/scm_mmhr/alter_relationship_first_met_year.sql`

- [ ] **Step 1: tb_cpi_index DDL + 초기 데이터**

스펙 Section 3.1의 DDL과 INSERT문 그대로 작성.

- [ ] **Step 2: tb_statistics_aggregate DDL**

스펙 Section 3.2의 DDL (유니크 인덱스 포함).

- [ ] **Step 3: tb_recommendation_default DDL + 초기 데이터**

스펙 Section 3.3의 DDL과 INSERT문.

- [ ] **Step 4: tb_budget DDL**

스펙 Section 3.5의 DDL (활성 유니크 제약 포함).

- [ ] **Step 5: ALTER 스크립트**

```sql
ALTER TABLE scm_mmhr.tb_relationship ADD COLUMN first_met_year INT;
ALTER TABLE scm_mmhr.tb_relationship_type ADD COLUMN category_code VARCHAR(20);
```

---

### Task 2: Backend — VO/DTO 생성

**Files (32_application 내):**
- Create: `vo/mmhr/CpiIndexVO.java`
- Create: `vo/mmhr/StatisticsAggregateVO.java`
- Create: `vo/mmhr/RecommendationDefaultVO.java`
- Create: `vo/mmhr/BudgetVO.java`
- Create: `recommendation/v1/dto/GetAmountRequestDtoV1.java`
- Create: `recommendation/v1/dto/GetAmountResponseDtoV1.java`
- Create: `recommendation/v1/dto/GetInflationRequestDtoV1.java`
- Create: `recommendation/v1/dto/GetInflationResponseDtoV1.java`
- Create: `recommendation/v1/dto/GetTrendResponseDtoV1.java`
- Create: `budget/v1/dto/GetBudgetStatusRequestDtoV1.java`
- Create: `budget/v1/dto/SaveBudgetRequestDtoV1.java`

- [ ] **Step 1: VO 클래스 4개 생성**

기존 프로젝트 패턴 따름 (`@Data @EqualsAndHashCode(callSuper = true)`, BaseVO 상속).

- [ ] **Step 2: Recommendation DTO 생성**

`GetAmountResponseDtoV1`는 스펙 Section 4.4의 JSON 구조를 Java 클래스로 변환. 중첩 클래스로 `PersonalRecommendation`, `AggregateRecommendation`, `FinalRecommendation`, `InflationInfo` 포함.

- [ ] **Step 3: Budget DTO 생성**

---

### Task 3: Backend — KoreanAmountRounder 유틸리티

**Files:**
- Create: `32_application/.../common/util/KoreanAmountRounder.java`

- [ ] **Step 1: 라운딩 유틸리티 구현**

```java
public class KoreanAmountRounder {

    private static final long[] LOW_LADDER = {10000, 30000, 50000, 70000};

    public static long round(long amount) {
        if (amount <= 0) return 10000;
        if (amount > 70000 && amount < 100000) return 100000;
        if (amount >= 100000) {
            return (long) Math.ceil(amount / 50000.0) * 50000;
        }
        // 10만 미만: 사다리에서 가장 가까운 값 (동일 거리 시 올림)
        long closest = LOW_LADDER[0];
        long minDiff = Long.MAX_VALUE;
        for (long step : LOW_LADDER) {
            long diff = Math.abs(amount - step);
            if (diff < minDiff || (diff == minDiff && step > closest)) {
                minDiff = diff;
                closest = step;
            }
        }
        return closest;
    }

    public static long getLowerOption(long amount) {
        List<Long> ladder = buildLadder(amount + 50000);
        int idx = ladder.indexOf(amount);
        return idx > 0 ? ladder.get(idx - 1) : -1; // -1 = 없음
    }

    public static long getUpperOption(long amount) {
        List<Long> ladder = buildLadder(amount + 100000);
        int idx = ladder.indexOf(amount);
        return idx >= 0 && idx < ladder.size() - 1 ? ladder.get(idx + 1) : amount + 50000;
    }

    private static List<Long> buildLadder(long upTo) {
        List<Long> ladder = new ArrayList<>(Arrays.asList(10000L, 30000L, 50000L, 70000L));
        for (long v = 100000; v <= upTo; v += 50000) {
            ladder.add(v);
        }
        return ladder;
    }
}
```

- [ ] **Step 2: RecommendationTypeMapper 유틸리티 구현**

```java
public class RecommendationTypeMapper {
    private static final Map<String, String> RECORD_TO_REC = Map.of(
        "WEDDING", "WEDDING", "FUNERAL", "FUNERAL",
        "BIRTHDAY", "BIRTHDAY", "ANNIVERSARY", "CELEBRATION", "OTHER", "OTHER"
    );
    private static final Map<String, String> CEREMONY_TO_REC = Map.of(
        "WEDDING", "WEDDING", "FUNERAL", "FUNERAL",
        "BIRTHDAY_PARTY", "BIRTHDAY", "CELEBRATION", "CELEBRATION", "OTHER", "OTHER"
    );

    public static String fromRecordType(String recordTypeCode) {
        return RECORD_TO_REC.getOrDefault(recordTypeCode, "OTHER");
    }
    public static String fromCeremonyType(String ceremonyTypeCode) {
        return CEREMONY_TO_REC.getOrDefault(ceremonyTypeCode, "OTHER");
    }
}
```

---

## Phase B: Backend 핵심 로직

### Task 4: Recommendation DAO + Mapper

**Files:**
- Create: `recommendation/v1/dao/RecommendationDaoV1.java`
- Create: `resources/mapper/recommendation/v1/RecommendationMapperV1.xml`

- [ ] **Step 1: DAO 인터페이스**

```java
@Mapper
public interface RecommendationDaoV1 {
    // 개인 추천용: 해당 관계의 동일 유형 과거 기록 조회
    List<RecordVO> selectPersonalRecords(@Param("userNo") Long userNo,
        @Param("relationshipNo") Long relationshipNo,
        @Param("recommendationType") String recommendationType,
        @Param("directionCode") String directionCode);

    // CPI 테이블 전체 조회
    List<CpiIndexVO> selectAllCpi();

    // 집단 통계 조회 (가장 최근 stat_date)
    StatisticsAggregateVO selectLatestAggregateStat(
        @Param("recommendationType") String recommendationType,
        @Param("directionCode") String directionCode,
        @Param("durationGroup") String durationGroup,
        @Param("genderCode") String genderCode,
        @Param("relationshipCategory") String relationshipCategory);

    // 글로벌 기본값 조회
    RecommendationDefaultVO selectDefault(
        @Param("recommendationType") String recommendationType,
        @Param("directionCode") String directionCode);

    // 관계 정보 (first_met_year, category_code 포함)
    RelationshipVO selectRelationshipWithCategory(@Param("relationshipNo") Long relationshipNo);

    // 트렌드: 최근 N개월 집단 통계
    List<StatisticsAggregateVO> selectTrendStats(
        @Param("recommendationType") String recommendationType,
        @Param("months") int months);
}
```

- [ ] **Step 2: Mapper XML 작성**

각 DAO 메서드에 대응하는 SELECT 쿼리 작성.

---

### Task 5: RecommendationService 구현

**Files:**
- Create: `recommendation/v1/service/RecommendationServiceV1.java`

- [ ] **Step 1: getAmount 메서드 구현**

```java
public GetAmountResponseDtoV1 getAmount(Long userNo, GetAmountRequestDtoV1 dto) {
    // 1. 관계 정보 조회 (first_met_year, category_code)
    // 2. 관계지속기간 그룹 산출 (first_met_year 없으면 reg_date 폴백)
    // 3. 개인 추천 계산
    //    - 과거 기록 조회 → CPI 보정 → 시간 가중 평균 → 기간 보정 계수
    // 4. 집단 추천 계산
    //    - tb_statistics_aggregate에서 조건 매칭 (gender, category 폴백)
    //    - N<30이면 null 폴백 (전체 통합 통계)
    // 5. 가중치 결정 (기록 건수 + 집단 N 기반)
    // 6. 통합 = 개인 × 가중치 + 집단 × 가중치
    // 7. KoreanAmountRounder.round() 적용
    // 8. lowerOption, upperOption 산출
    // 9. 물가 환산 정보 (마지막 기록 기준)
    // 10. 응답 조립
}
```

- [ ] **Step 2: getInflationValue 메서드 구현**

```java
public GetInflationResponseDtoV1 getInflationValue(GetInflationRequestDtoV1 dto) {
    List<CpiIndexVO> cpiList = dao.selectAllCpi();
    Map<Integer, CpiIndexVO> cpiMap = cpiList.stream().collect(toMap(CpiIndexVO::getYear, v -> v));

    // CPI 기준
    BigDecimal fromCpi = cpiMap.get(dto.getFromYear()).getCpiValue();
    BigDecimal toCpi = getOrEstimateCpi(cpiMap, dto.getToYear());
    BigDecimal byCpi = dto.getAmount().multiply(toCpi).divide(fromCpi, 0, RoundingMode.HALF_UP);

    // 예금 이자율 복리
    BigDecimal byDeposit = dto.getAmount();
    for (int y = dto.getFromYear() + 1; y <= dto.getToYear(); y++) {
        BigDecimal rate = cpiMap.containsKey(y) ? cpiMap.get(y).getAvgDepositRate() : lastKnownRate;
        byDeposit = byDeposit.multiply(BigDecimal.ONE.add(rate.divide(BigDecimal.valueOf(100))));
    }

    // 사용자 지정 이자율 복리
    BigDecimal byCustom = null;
    if (dto.getCustomRate() != null) {
        int years = dto.getToYear() - dto.getFromYear();
        byCustom = dto.getAmount().multiply(
            BigDecimal.ONE.add(dto.getCustomRate().divide(BigDecimal.valueOf(100)))
                .pow(years));
    }

    // 응답 조립
}
```

- [ ] **Step 3: getTrend 메서드 구현**

최근 12개월 집단 통계 변화 추이 + 개인 vs 집단 비교.

---

### Task 6: RecommendationController + BudgetController

**Files:**
- Create: `recommendation/controller/RecommendationController.java`
- Create: `budget/controller/BudgetController.java`
- Create: `budget/v1/service/BudgetServiceV1.java`
- Create: `budget/v1/dao/BudgetDaoV1.java`
- Create: `resources/mapper/budget/v1/BudgetMapper*.xml`

- [ ] **Step 1: RecommendationController**

3개 엔드포인트: `/getAmount`, `/getInflationValue`, `/getTrend`

- [ ] **Step 2: BudgetController + Service + DAO**

3개 엔드포인트: `/getStatus`, `/save`, `/delete`
UPSERT 패턴 (활성 유니크 제약 기반).

---

### Task 7: Spring Scheduler (33_system)

**Files:**
- Create: `33_system/.../scheduler/StatisticsAggregateScheduler.java`
- Create: `33_system/.../scheduler/CpiUpdateScheduler.java`

- [ ] **Step 1: StatisticsAggregateScheduler**

```java
@Scheduled(cron = "0 0 3 * * *")  // 매일 새벽 3시
public void buildDailyAggregateStatistics() {
    // 1. SELECT: 전체 사용자 tb_record JOIN tb_relationship JOIN tb_relationship_type JOIN tb_user
    //    - record_type_code → recommendation_type 변환
    //    - first_met_year → duration_group (null이면 reg_date 폴백)
    //    - category_code → relationship_category
    //    - gender_code
    // 2. GROUP BY: recommendation_type × direction × duration × gender × category
    //    + gender=null, category=null 전체 통합도 생성
    // 3. 각 그룹별 avg, median, min, max, p25, p75 산출
    // 4. UPSERT (INSERT ON CONFLICT UPDATE) into tb_statistics_aggregate
    // 5. Cleanup: 90일 이상 MONTHLY, 5년 이상 YEARLY 삭제
}
```

- [ ] **Step 2: CpiUpdateScheduler**

```java
@Scheduled(cron = "0 0 4 1 * *")  // 매월 1일
public void updateCpiAndDepositRate() {
    // 현재 연도 CPI 미확정 시: 직전 2년 평균 상승률로 추정
    // data_confirmed = 'N' 설정
}
```

---

## Phase C: Frontend 핵심

### Task 8: hs-common 타입 + API

**Files:**
- Create: `21_hs-common/src/types/recommendation.ts`
- Create: `21_hs-common/src/types/budget.ts`
- Modify: `21_hs-common/src/types/relationship.ts` (first_met_year 추가)
- Modify: `21_hs-common/src/types/index.ts`
- Create: `21_hs-common/src/api/entities/recommendationApi.ts`
- Create: `21_hs-common/src/api/entities/budgetApi.ts`

- [ ] **Step 1: recommendation.ts 타입**

스펙 Section 4.4, 4.5의 응답 구조를 TypeScript interface로 변환.

- [ ] **Step 2: budget.ts 타입**

- [ ] **Step 3: relationship.ts에 first_met_year 추가**

`Relationship` interface에 `firstMetYear?: number` 추가.
`CreateRelationshipRequest`, `UpdateRelationshipRequest`에도 추가.

- [ ] **Step 4: API 클라이언트 2개 생성**

기존 factory 패턴 따름.

---

### Task 9: 관계 폼 first_met_year 필드

**Files:**
- Modify: `31_webadmin/src/features/relationship/ui/RelationshipForm.tsx`

- [ ] **Step 1: Zod 스키마에 firstMetYear 추가**

```ts
firstMetYear: z.number().min(1950).max(new Date().getFullYear()).optional()
```

- [ ] **Step 2: 폼 UI에 연도 입력 필드 추가**

"처음 만난 연도" Input (type number, 4자리).

---

### Task 10: 추천 카드 컴포넌트 + 폼 통합

**Files:**
- Create: `31_webadmin/src/entities/recommendation/api/recommendationApi.ts`
- Create: `31_webadmin/src/entities/recommendation/model/types.ts`
- Create: `31_webadmin/src/entities/recommendation/model/queries.ts`
- Create: `31_webadmin/src/features/recommendation/ui/RecommendationCard.tsx`
- Modify: `31_webadmin/src/features/record/ui/RecordForm.tsx`
- Modify: `31_webadmin/src/features/ceremony/ui/CeremonyForm.tsx`

- [ ] **Step 1: entities 레이어 생성**

API 인스턴스 + TanStack Query hook (`useGetRecommendation`).

- [ ] **Step 2: RecommendationCard 컴포넌트**

스펙 Section 5.1의 UI. 추천 금액 + lower/upper + 개인/집단 설명 + 물가 정보 + "추천대로 적용" 버튼.

- [ ] **Step 3: RecordForm에 통합**

관계 상세 페이지에서 열릴 때 `relationshipNo`가 있으면, 유형+방향 선택 시 `useGetRecommendation` 호출 → RecommendationCard 표시 → "적용" 시 금액 자동 입력.

- [ ] **Step 4: CeremonyForm에 통합**

`relationshipNo`가 있을 때만 추천 카드 표시.

---

### Task 11: 물가 환산 계산기 페이지

**Files:**
- Create: `31_webadmin/src/app/(authenticated)/tools/inflation-calculator/page.tsx`
- Modify: `31_webadmin/src/shared/config/menu-config.ts` (도구 메뉴)
- Modify: `31_webadmin/src/app/router.tsx`

- [ ] **Step 1: 계산기 페이지 구현**

금액/기준연도/환산연도/커스텀이자율 입력 → 3가지 결과 표시 (CPI/예금이자/커스텀).
React Hook Form으로 입력 관리. 결과는 카드 형태로 표시.

- [ ] **Step 2: 메뉴 + 라우트**

menu-config에 "도구" 그룹 추가 (Wrench 아이콘, children: 물가환산, 예산플래너).
router에 `/tools/inflation-calculator` 라우트 추가.

---

## Phase D: 편의 기능

### Task 12: 예산 플래너

**Files:**
- Create: `31_webadmin/src/entities/budget/api/budgetApi.ts`
- Create: `31_webadmin/src/entities/budget/model/types.ts`
- Create: `31_webadmin/src/entities/budget/model/queries.ts`
- Create: `31_webadmin/src/features/budget/ui/BudgetForm.tsx`
- Create: `31_webadmin/src/app/(authenticated)/tools/budget/page.tsx`
- Create: `31_webadmin/src/widgets/dashboard/BudgetWidget.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/page.tsx` (대시보드에 위젯)

- [ ] **Step 1: entities + features 레이어**
- [ ] **Step 2: 예산 설정 페이지**

월별/연별 예산 설정 + 지출 추이 Recharts 차트.

- [ ] **Step 3: BudgetWidget 대시보드 위젯**

프로그레스 바 + 다가오는 예상 지출 미리보기.

---

### Task 13: 관계 타임라인

**Files:**
- Create: `31_webadmin/src/widgets/relationship/RelationshipTimeline.tsx`
- Modify: 관계 상세 페이지 (탭 추가)

- [ ] **Step 1: RelationshipTimeline 컴포넌트**

시간순 거래 히스토리 + 물가보정 금액 + 관계 건강도(균형→초록, 불균형→빨강).

- [ ] **Step 2: 관계 상세 페이지에 타임라인 탭 추가**

---

### Task 14: BalanceWidget 물가보정 + 트렌드 인사이트

**Files:**
- Modify: `31_webadmin/src/widgets/dashboard/BalanceWidget.tsx`
- Create: `31_webadmin/src/widgets/dashboard/TrendInsightCard.tsx`
- Modify: `31_webadmin/src/app/(authenticated)/page.tsx`

- [ ] **Step 1: BalanceWidget에 물가보정 토글 추가**

ON/OFF 토글 → ON 시 CPI 기반 실질 잔액 표시.

- [ ] **Step 2: TrendInsightCard 컴포넌트**

집단 통계 기반 인사이트 카드 (전년 대비 변화, 개인 vs 평균 비교).

- [ ] **Step 3: 대시보드에 추가**

---

### Task 15: 스마트 알림 + Quick Entry

**Files:**
- Modify: `31_webadmin/src/widgets/header/NotificationDropdown.tsx`
- Modify: 알림 카드 컴포넌트

- [ ] **Step 1: 알림 카드에 추천 금액 포함**

D-day 알림에 추천 금액 표시 + "바로 기록" 버튼.

- [ ] **Step 2: Quick Entry 동작**

"바로 기록" 클릭 → 관계+유형+추천금액 미리 채워진 RecordForm 열기.

---

## 검증

- [ ] **Frontend type-check:** `cd 31_webadmin && npm run type-check`
- [ ] **Frontend build:** `cd 31_webadmin && npm run build`
- [ ] **Backend build:** `cd 32_application && ./gradlew build`
- [ ] **System build:** `cd 33_system && ./gradlew build`
- [ ] **hs-common 변경 영향:** webadmin + appuser 타입체크
