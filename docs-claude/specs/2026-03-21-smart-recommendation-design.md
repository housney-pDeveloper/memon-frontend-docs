# MEMON 스마트 금액 추천 엔진 + 편의 기능 설계

**작성일:** 2026-03-21
**범위:** Frontend + Backend + Database

---

## 1. 개요

MEMON을 단순 경조사 기록 관리에서 **경조사 재무 어드바이저**로 포지셔닝하는 핵심 차별화 기능.

3단계 추천 파이프라인(개인 데이터 → 집단 통계 → 통합 추천)과 한국 정서 라운딩을 통해 합리적 경조사 금액을 제안하고, 물가 환산 / 예산 플래너 / 스마트 알림 등 편의 기능을 제공한다.

---

## 2. 금액 추천 엔진

### 2.1 3단계 추천 파이프라인

```
[1단계: 개인 데이터 기반]
  └→ 사용자의 해당 관계 과거 기록 + CPI 물가보정 → 개인 추천 금액

[2단계: 집단 통계 기반]
  └→ 전체 사용자 익명 통계 (성별/관계유형/관계지속햇수/경조사유형별) → 평균 추천 금액

[3단계: 통합 추천]
  └→ 개인 + 집단 가중 합산 → 한국 정서 라운딩 → 최종 추천 금액
```

### 2.2 가중치 전략

| 조건 | 개인 가중치 | 집단 가중치 |
|------|-----------|-----------|
| 해당 관계와 기록 3건 이상 | 70% | 30% |
| 해당 관계와 기록 1-2건 | 40% | 60% |
| 해당 관계와 기록 없음 | 10% | 90% |
| 전체 사용자 통계 부족 (N<30) | 100% | 0% |

**우선순위 규칙:** 두 조건이 동시에 해당될 때 (예: 개인 기록 0건 AND 집단 N<30):
- 집단 N<30이면서 개인 기록도 0건 → **글로벌 기본값 테이블** 사용 (경조사유형별 고정 중앙값)
- 글로벌 기본값도 없으면 → `confidence: "INSUFFICIENT"`, 추천 불가 메시지 표시

**신뢰도(confidence) 정의:**
- `HIGH`: 개인 기록 ≥ 3건 AND 집단 N ≥ 30
- `MEDIUM`: 개인 기록 1-2건 OR 집단 N ≥ 30
- `LOW`: 개인 기록 0건 AND 집단 N < 30 (글로벌 기본값 사용)
- `INSUFFICIENT`: 데이터 없음

### 2.3 recordTypeCode / ceremonyTypeCode 통합 매핑

두 도메인의 타입 코드 차이를 통합하는 매핑 테이블:

| 통합 코드 (recommendation_type) | recordTypeCode | ceremonyTypeCode |
|-------------------------------|----------------|-----------------|
| `WEDDING` | WEDDING | WEDDING |
| `FUNERAL` | FUNERAL | FUNERAL |
| `BIRTHDAY` | BIRTHDAY | BIRTHDAY_PARTY |
| `CELEBRATION` | ANNIVERSARY | CELEBRATION |
| `OTHER` | OTHER | OTHER |

- `tb_statistics_aggregate`에서는 통합 코드(`recommendation_type`)를 사용
- Scheduler 집계 시 `tb_record.record_type_code`를 통합 코드로 변환
- 추천 API 요청에서는 `recommendationType` 필드 사용 (NOT ceremonyTypeCode)
- RecordForm/CeremonyForm에서 해당 폼의 typeCode를 통합 코드로 변환하여 API 호출

### 2.4 한국 정서 라운딩 로직

**표준 금액 사다리:**
```
1만 → 3만 → 5만 → 7만 → 10만 → 15만 → 20만 → 25만 → 30만 → 35만 → 40만 → 45만 → 50만 → ...
```

**규칙:**
- 1만~7만: 홀수 만원 단위 (1, 3, 5, 7)
- 9만원 없음 (아홉수 회피)
- **7만 초과 ~ 10만 미만 → 무조건 10만으로 올림** (한국 정서상 8~9만원대 사용 안 함)
- 10만 이상 → 5만 단위로 증가
- 동일 거리 → **올림** (한국 관습상 넉넉하게 주는 것이 예절)
- 최종 추천과 함께 "↓ 한 단계" / "↑ 한 단계" 인접 옵션도 표시
- 경계값: 최저 = 1만 (lowerOption: null), 최고 = 제한 없음 (upperOption 항상 존재)

**라운딩 알고리즘:**
```
function roundToKoreanStandard(amount):
  // 7만 초과 ~ 10만 미만: 강제 올림
  if amount > 70000 and amount < 100000:
    return 100000

  // 10만 이상: 5만 단위 올림
  if amount >= 100000:
    return ceil(amount / 50000) * 50000

  // 10만 미만: 사다리에서 가장 가까운 값 (동일 거리 시 올림)
  ladder = [10000, 30000, 50000, 70000]
  return closestCeil(ladder, amount)

function getAdjacentOptions(amount):
  fullLadder = [10000, 30000, 50000, 70000, 100000, 150000, ...]
  idx = fullLadder.indexOf(amount)
  lower = idx > 0 ? fullLadder[idx - 1] : null
  upper = fullLadder[idx + 1]
  return { lower, upper }
```

### 2.5 개인 추천 계산 로직

```
1. 해당 관계(relationshipNo)와의 과거 기록 중 동일 recommendationType + directionCode 필터
2. 각 기록의 금액을 CPI 보정하여 현재가치로 환산:
   presentValue = pastAmount × (currentCPI / pastCPI)
3. 보정된 금액들의 가중 평균 산출 (최근 기록에 더 높은 가중치)
   - 시간 가중: 1년 이내 ×1.5, 1-3년 ×1.2, 3-5년 ×1.0, 5년+ ×0.8
4. 관계 지속기간(durationGroup)에 따른 보정 계수 적용:
   - 0-2Y: ×0.9 (새 관계는 소폭 낮게)
   - 3-5Y: ×1.0 (기본)
   - 6-10Y: ×1.05
   - 10Y+: ×1.1 (오래된 관계는 소폭 높게)
```

**관계지속기간 산출:**
- `first_met_year`가 있으면: `현재년도 - first_met_year`
- `first_met_year`가 null이면: `현재년도 - YEAR(reg_date)` (관계 등록일 폴백)

### 2.6 집단 추천 계산 로직

```
1. tb_statistics_aggregate에서 조건 매칭:
   - recommendation_type 일치
   - direction_code 일치
   - relationship_duration_group 일치
   - gender_code 일치 (없으면 null 통합 통계 폴백)
   - relationship_category 일치 (없으면 null 통합 통계 폴백)
2. 중앙값(median_amount)을 기본 추천으로 사용 (평균은 극단값 영향 큼)
3. 표본 수(sample_count) < 30이면 신뢰도 LOW
```

### 2.7 예금 이자율 복리 계산 공식

```
// CPI 기준
presentValueByCpi = pastAmount × (currentYearCPI / pastYearCPI)

// 예금 이자율 복리 (연간 이자율 product)
presentValueByDeposit = pastAmount × ∏(1 + rate_i/100) for each year i from pastYear+1 to currentYear
  (rate_i = 해당 연도의 avg_deposit_rate)

// 사용자 지정 이자율 복리
presentValueByCustom = pastAmount × (1 + customRate/100)^(currentYear - pastYear)
```

---

## 3. Database 설계

### 3.1 CPI 물가지수 + 예금 이자율 테이블

```sql
CREATE TABLE scm_mmhr.tb_cpi_index (
    year            INT             PRIMARY KEY,
    cpi_value       DECIMAL(10,2)   NOT NULL,
    base_year       INT             NOT NULL DEFAULT 2020,
    avg_deposit_rate DECIMAL(5,2),
    data_confirmed  CHAR(1)         DEFAULT 'Y',  -- N이면 추정치
    reg_date        TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    mod_date        TIMESTAMP       DEFAULT CURRENT_TIMESTAMP
);

-- 초기 데이터
INSERT INTO scm_mmhr.tb_cpi_index (year, cpi_value, base_year, avg_deposit_rate) VALUES
(2000, 58.41, 2020, 7.08),
(2005, 69.55, 2020, 3.58),
(2010, 79.07, 2020, 3.18),
(2015, 89.85, 2020, 1.66),
(2018, 95.44, 2020, 1.83),
(2019, 96.83, 2020, 1.74),
(2020, 100.00, 2020, 0.97),
(2021, 102.50, 2020, 0.85),
(2022, 107.71, 2020, 2.76),
(2023, 111.56, 2020, 3.54),
(2024, 114.41, 2020, 3.32),
(2025, 116.80, 2020, 2.95);
```

**현재 연도 CPI 미확정 처리:**
- 현재 연도의 CPI가 없으면: 직전 연도 CPI × (직전 2년간 평균 상승률) 로 추정
- `data_confirmed = 'N'`으로 표시 → Frontend에서 "추정치" 경고 표시
- `dataAvailableToYear` 필드를 응답에 포함

### 3.2 집단 통계 테이블

```sql
CREATE TABLE scm_mmhr.tb_statistics_aggregate (
    stat_no                     BIGSERIAL       PRIMARY KEY,
    stat_date                   DATE            NOT NULL,
    stat_type                   VARCHAR(20)     NOT NULL,   -- MONTHLY / YEARLY
    recommendation_type         VARCHAR(20)     NOT NULL,   -- WEDDING / FUNERAL / BIRTHDAY / CELEBRATION / OTHER (통합 코드)
    direction_code              VARCHAR(20)     NOT NULL,   -- GIVEN / RECEIVED
    relationship_duration_group VARCHAR(20)     NOT NULL,   -- 0-2Y / 3-5Y / 6-10Y / 10Y+
    gender_code                 VARCHAR(10),                -- MALE / FEMALE / null(전체)
    relationship_category       VARCHAR(20),                -- FAMILY / FRIEND / COLLEAGUE / ACQUAINTANCE / null(전체)
    sample_count                INT             NOT NULL DEFAULT 0,
    avg_amount                  DECIMAL(12,2),
    median_amount               DECIMAL(12,2),
    min_amount                  DECIMAL(12,2),
    max_amount                  DECIMAL(12,2),
    percentile_25               DECIMAL(12,2),
    percentile_75               DECIMAL(12,2),
    reg_date                    TIMESTAMP       DEFAULT CURRENT_TIMESTAMP
);

-- 비즈니스 키 유니크 제약 (UPSERT 지원)
CREATE UNIQUE INDEX idx_stat_agg_unique ON scm_mmhr.tb_statistics_aggregate(
    stat_type, recommendation_type, direction_code,
    relationship_duration_group,
    COALESCE(gender_code, '_ALL_'),
    COALESCE(relationship_category, '_ALL_'),
    stat_date
);

CREATE INDEX idx_stat_agg_lookup ON scm_mmhr.tb_statistics_aggregate(
    stat_type, recommendation_type, direction_code, relationship_duration_group
);
```

**데이터 보존 정책:**
- MONTHLY: 최근 90일만 유지
- YEARLY: 5년간 유지
- Scheduler에 cleanup 단계 포함

### 3.3 글로벌 기본값 테이블

```sql
CREATE TABLE scm_mmhr.tb_recommendation_default (
    default_no          BIGSERIAL   PRIMARY KEY,
    recommendation_type VARCHAR(20) NOT NULL,
    direction_code      VARCHAR(20) NOT NULL,
    default_amount      DECIMAL(12,2) NOT NULL,
    description         VARCHAR(200),
    reg_date            TIMESTAMP   DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(recommendation_type, direction_code)
);

-- 초기 데이터 (한국 경조사 일반적 금액)
INSERT INTO scm_mmhr.tb_recommendation_default VALUES
(DEFAULT, 'WEDDING', 'GIVEN', 50000, '결혼 축의금 일반'),
(DEFAULT, 'FUNERAL', 'GIVEN', 50000, '조의금 일반'),
(DEFAULT, 'BIRTHDAY', 'GIVEN', 30000, '생일 축하금'),
(DEFAULT, 'CELEBRATION', 'GIVEN', 30000, '축하금'),
(DEFAULT, 'OTHER', 'GIVEN', 30000, '기타');
```

### 3.4 관계 테이블 확장

```sql
ALTER TABLE scm_mmhr.tb_relationship
    ADD COLUMN first_met_year INT;

ALTER TABLE scm_mmhr.tb_relationship_type
    ADD COLUMN category_code VARCHAR(20);
    -- FAMILY / FRIEND / COLLEAGUE / ACQUAINTANCE
```

### 3.5 경조사 예산 테이블

```sql
CREATE TABLE scm_mmhr.tb_budget (
    budget_no       BIGSERIAL       PRIMARY KEY,
    user_no         BIGINT          NOT NULL,
    budget_type     VARCHAR(20)     NOT NULL,   -- MONTHLY / YEARLY
    budget_year     INT             NOT NULL,
    budget_month    INT,
    budget_amount   DECIMAL(12,2)   NOT NULL,
    reg_date        TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    mod_date        TIMESTAMP       DEFAULT CURRENT_TIMESTAMP,
    deleted_yn      CHAR(1)         DEFAULT 'N',
    CONSTRAINT fk_budget_user FOREIGN KEY (user_no) REFERENCES scm_mmhr.tb_user(user_no)
);

-- 활성 예산 유니크 제약
CREATE UNIQUE INDEX idx_budget_unique_active
ON scm_mmhr.tb_budget(user_no, budget_type, budget_year, COALESCE(budget_month, 0))
WHERE deleted_yn = 'N';
```

---

## 4. Backend 설계

> **참고:** 모든 API에서 `user_no`는 Gateway가 주입한 JWT 토큰에서 `RequestContextHolder`를 통해 추출. Request body에 포함하지 않음.

### 4.1 Spring Scheduler (33_system)

**Scheduler 1: 매월 1일 새벽 3시 — 집단 통계 집계**
```java
@Scheduled(cron = "0 0 3 * * *")  // 매일
public void buildDailyAggregateStatistics() {
    // 1. 전체 사용자의 tb_record를 조인하여 집계
    //    - record_type_code → recommendation_type 변환 (매핑 테이블)
    //    - tb_relationship.first_met_year → 관계지속기간 그룹 (null이면 reg_date 폴백)
    //    - tb_relationship_type.category_code → 관계 카테고리
    //    - tb_user.gender_code → 성별
    // 2. GROUP BY: recommendation_type × direction_code × duration_group × gender × category
    //    + gender=null, category=null인 전체 통합 통계도 생성
    // 3. 각 그룹별 avg, median, min, max, p25, p75 산출
    // 4. UPSERT into tb_statistics_aggregate (stat_type = 'MONTHLY')
    // 5. 매년 1월에는 stat_type = 'YEARLY'도 생성
    // 6. 90일 이상 된 MONTHLY, 5년 이상 된 YEARLY 데이터 정리
}
```

**Scheduler 2: 매월 1일 새벽 4시 — CPI/이자율 갱신**
```java
@Scheduled(cron = "0 0 4 1 * *")
public void updateCpiAndDepositRate() {
    // 현재: 수동 관리 테이블 체크만 수행
    // 향후: 한국은행 경제통계 API 연동 (인터페이스 분리하여 교체 가능)
    // 현재 연도 CPI 미확정 시: 직전 2년 평균 상승률로 추정, data_confirmed='N'
}
```

### 4.2 추천 API (32_application — RecommendationController)

**Base Path:** `/recommendation`

| Endpoint | 설명 | Request |
|---------|------|---------|
| `POST /getAmount` | 금액 추천 | `{ relationshipNo, recommendationType, directionCode }` |
| `POST /getInflationValue` | 물가 환산 | `{ amount, fromYear, toYear, customRate? }` |
| `POST /getTrend` | 트렌드 인사이트 | `{ recommendationType?, year? }` |

### 4.3 예산 API (32_application — BudgetController)

**Base Path:** `/budget`

| Endpoint | 설명 | Request |
|---------|------|---------|
| `POST /getStatus` | 예산 현황 | `{ budgetType, year, month? }` |
| `POST /save` | 예산 설정 | `{ budgetType, year, month?, amount }` |
| `POST /delete` | 예산 삭제 | `{ budgetNo }` |

### 4.4 `POST /recommendation/getAmount` 응답

```json
{
  "personalRecommendation": {
    "rawAmount": 82000,
    "adjustedAmount": 95000,
    "basedOnRecordCount": 5,
    "durationGroup": "6-10Y",
    "description": "과거 5건의 유사 기록 CPI 보정 평균"
  },
  "aggregateRecommendation": {
    "avgAmount": 78000,
    "medianAmount": 70000,
    "sampleCount": 1240,
    "durationGroup": "6-10Y",
    "genderCode": "MALE",
    "relationshipCategory": "FRIEND",
    "description": "남성, 친구, 6-10년 관계, 결혼식 중앙값"
  },
  "finalRecommendation": {
    "amount": 100000,
    "lowerOption": 70000,
    "upperOption": 150000,
    "confidence": "HIGH",
    "personalWeight": 0.7,
    "aggregateWeight": 0.3,
    "reasoning": "과거 유사 기록 평균 95,000원 + 전체 사용자 중앙값 70,000원 기반"
  },
  "inflationInfo": {
    "lastGivenAmount": 50000,
    "lastGivenYear": 2018,
    "presentValueByCpi": 58500,
    "presentValueByDeposit": 56200,
    "cpiInflationRate": 17.0,
    "depositAccumulatedRate": 12.4,
    "dataConfirmed": true,
    "dataAvailableToYear": 2025
  }
}
```

### 4.5 `POST /recommendation/getInflationValue` 응답

```json
{
  "originalAmount": 50000,
  "fromYear": 2018,
  "toYear": 2026,
  "byCpi": {
    "presentValue": 58500,
    "totalRate": 17.0,
    "description": "소비자물가지수(CPI) 기준",
    "dataConfirmed": true
  },
  "byDepositRate": {
    "presentValue": 56200,
    "totalRate": 12.4,
    "formula": "복리: 50000 × Π(1 + rate_i/100)",
    "description": "은행 평균 예금 이자율 기준 복리"
  },
  "byCustomRate": {
    "customRate": 5.0,
    "presentValue": 73900,
    "totalRate": 47.7,
    "formula": "50000 × (1 + 0.05)^8",
    "description": "사용자 지정 이자율 5% 복리"
  },
  "dataAvailableToYear": 2025
}
```

### 4.6 Spring Cache 적용

집단 통계 데이터는 월 1회 갱신이므로 Application Server에서 캐싱:
```java
@Cacheable(value = "aggregateStats", key = "#type + '_' + #direction + '_' + #durationGroup + '_' + #gender + '_' + #category")
public StatisticsAggregateVO getAggregateStats(String type, String direction, String durationGroup, String gender, String category) { ... }
```
- TTL: 24시간 (매일 자정에 evict)

---

## 5. Frontend 설계

### 5.1 금액 추천 카드

**위치:** RecordForm / CeremonyForm 내부

**동작:**
- RecordForm: `relationshipNo`는 상위 컨텍스트(관계 상세 페이지)에서 전달됨 → 유형+방향 선택 시 자동 추천
- CeremonyForm: `relationshipNo`가 optional → 있으면 추천 표시, 없으면 집단 통계만 표시
- "추천대로 적용" 클릭 시 금액 필드에 자동 입력

**FSD 레이어:**
- `entities/recommendation/api/` — getAmount, getInflationValue, getTrend (읽기)
- `entities/recommendation/model/types.ts` — 타입 정의
- `entities/budget/api/` — getStatus (읽기)
- `features/budget/api/` — save, delete (쓰기)
- `features/recommendation/ui/RecommendationCard.tsx` — 추천 카드 UI

### 5.2 물가 환산 계산기

**위치 1:** 기록 상세 페이지 — 과거 기록 옆에 "현재 가치: XX원 (CPI +YY%)" 자동 표시
**위치 2:** `/tools/inflation-calculator` — 금액/연도/이자율 직접 입력 계산기

3가지 결과 동시 표시:
- CPI 기준 현재가치
- 예금이자율 복리 기준 현재가치
- 사용자 지정 이자율 복리 기준 현재가치

### 5.3 경조사 예산 플래너

**대시보드 위젯:** 이번 달 예산 프로그레스 바 + 다가오는 예상 지출 미리보기
**별도 페이지 `/tools/budget`:** 월별/연별 예산 설정 + 지출 추이 차트

### 5.4 스마트 알림 (추천 금액 포함)

D-day 알림에 추천 금액 포함 + "바로 기록 등록" 버튼 (Quick Entry)

### 5.5 관계 균형 대시보드 (물가보정)

기존 BalanceWidget에 물가보정 ON/OFF 토글 추가.
ON 시 CPI 기반으로 과거 금액을 현재가치로 환산하여 실질 잔액 표시.

### 5.6 경조사 트렌드 인사이트

통계 페이지 하단 — 집단 통계 기반:
- "올해 결혼식 평균 축의금: 12만원 (작년 대비 +8%)"
- "당신의 경조사비는 평균 대비 +25%"

### 5.7 빠른 기록 등록 (Quick Entry)

알림/추천에서 원터치로 기록 등록. 관계 + 유형 + 추천 금액이 미리 채워진 RecordForm.

### 5.8 관계 타임라인

관계 상세 페이지 새 탭 — 시간순 거래 히스토리 + 물가보정 금액 + 관계 건강도 시각화.

### 5.9 Sidebar 메뉴 구성

```
대시보드
관계관리
관계연결
기록관리
경조사관리
일정관리
통계
도구 ← 신규
  └ 물가환산 계산기
  └ 예산 플래너
```

---

## 6. 영향 파일

### Database
- `tb_cpi_index.sql` (신규)
- `tb_statistics_aggregate.sql` (신규)
- `tb_recommendation_default.sql` (신규)
- `tb_budget.sql` (신규)
- `tb_relationship` ALTER (first_met_year)
- `tb_relationship_type` ALTER (category_code)

### Backend (32_application)
- `RecommendationController.java` (신규)
- `RecommendationServiceV1.java` (신규)
- `RecommendationDaoV1.java` (신규)
- `KoreanAmountRounder.java` (신규 — 라운딩 유틸리티)
- `RecommendationMapper*.xml` (신규)
- `BudgetController.java` (신규)
- `BudgetServiceV1.java` (신규)
- `BudgetDaoV1.java` (신규)
- VO/DTO 클래스 다수

### Backend (33_system)
- `StatisticsAggregateScheduler.java` (신규)
- `CpiUpdateScheduler.java` (신규)
- `RecommendationTypeMapper.java` (신규 — recordTypeCode↔통합코드 변환)

### Frontend (21_hs-common)
- `types/recommendation.ts` (신규)
- `types/budget.ts` (신규)
- `types/relationship.ts` (first_met_year 추가)
- `api/entities/recommendationApi.ts` (신규)
- `api/entities/budgetApi.ts` (신규)

### Frontend (31_webadmin)
- `entities/recommendation/` (신규 — api, model)
- `entities/budget/` (신규 — api, model)
- `features/recommendation/ui/RecommendationCard.tsx` (신규)
- `features/budget/ui/BudgetForm.tsx` (신규)
- `widgets/dashboard/BudgetWidget.tsx` (신규)
- `widgets/dashboard/TrendInsightCard.tsx` (신규)
- `widgets/relationship/RelationshipTimeline.tsx` (신규)
- `app/(authenticated)/tools/inflation-calculator/page.tsx` (신규)
- `app/(authenticated)/tools/budget/page.tsx` (신규)
- RecordForm, CeremonyForm 수정 (추천 카드 통합)
- 관계 폼 수정 (first_met_year 필드)
- BalanceWidget 수정 (물가보정 토글)
- menu-config (도구 메뉴 추가)
- router (라우트 추가)

---

## 7. 구현 순서

### Phase A: 기반 인프라
1. DDL 생성 (tb_cpi_index, tb_statistics_aggregate, tb_recommendation_default, tb_budget, ALTER)
2. CPI 초기 데이터 INSERT
3. 글로벌 기본값 INSERT
4. 관계 유형 category_code 설정

### Phase B: Backend 핵심
5. KoreanAmountRounder 유틸리티
6. RecommendationTypeMapper (코드 변환)
7. 개인 추천 로직 (RecommendationServiceV1)
8. 집단 추천 로직 (RecommendationServiceV1)
9. 통합 추천 + API (RecommendationController)
10. 물가 환산 API
11. 예산 CRUD API (BudgetController)
12. 집단 통계 Scheduler
13. CPI 갱신 Scheduler

### Phase C: Frontend 핵심
14. hs-common 타입 + API 클라이언트
15. 관계 폼 first_met_year 필드
16. RecordForm/CeremonyForm 추천 카드 통합
17. 물가 환산 계산기 페이지

### Phase D: 편의 기능
18. 예산 플래너 (페이지 + 대시보드 위젯)
19. 관계 타임라인
20. BalanceWidget 물가보정 토글
21. 트렌드 인사이트 카드
22. 스마트 알림 + Quick Entry
23. 도구 메뉴 + 라우트
