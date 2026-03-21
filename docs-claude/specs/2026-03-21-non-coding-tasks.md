# MEMON 코딩 외 필요 작업 목록

> 작성일: 2026-03-21
> 대상: 서비스 출시를 위해 코드 외적으로 처리해야 하는 외부 API 키, 계정 등록, 법적 문서, 인프라 설정 등

---

## 1. 외부 API 키 / 계정

### Google AdSense

- [ ] Google AdSense 계정 생성 및 사이트 승인 신청
- [ ] 광고 단위(Ad Unit) 3개 생성
  - `side-left`: 160×600 (Wide Skyscraper)
  - `side-right`: 160×600 (Wide Skyscraper)
  - `top-banner`: 728×90 (Leaderboard)
- [ ] Publisher ID (`ca-pub-XXXXXXX`) 발급 후 환경 변수에 등록
- [ ] 각 광고 단위의 `data-ad-slot` 값 3개 발급 후 코드에 반영
- [ ] `ads.txt` 파일을 웹 루트(`/`)에 배치
- [ ] 승인 선행 조건 충족
  - 서비스에 충분한 콘텐츠 (로그인 없이 접근 가능한 공개 페이지)
  - 개인정보처리방침 페이지 게시
  - 이용약관 페이지 게시
  - 쿠키 동의 배너 구현

### OAuth 인증 (소셜 로그인)

- [ ] Google OAuth 2.0 Client ID / Client Secret 발급
  - Google Cloud Console → API 및 서비스 → 사용자 인증 정보
  - 승인된 콜백 URL 등록: `https://memon.kr/auth/callback`
- [ ] Kakao OAuth App Key 발급
  - Kakao Developers → 내 애플리케이션 등록
  - 플랫폼 Web 사이트 도메인 등록
  - Redirect URI 등록: `https://memon.kr/auth/callback`

### 이메일 발송 (관계연결 초대)

- [ ] 이메일 발송 서비스 계정 생성 (아래 중 선택)
  - AWS SES: 발신 도메인 인증 (SPF, DKIM, DMARC 레코드 설정)
  - SendGrid: API Key 발급
  - Mailgun: 도메인 인증 + API Key 발급
  - NHN Cloud Notification: 서비스 등록
- [ ] 발신 이메일 주소 확정 및 DNS 인증 (예: `noreply@memon.kr`)
- [ ] 관계연결 초대 이메일 HTML 템플릿 디자인

### 한국은행 경제통계 API (CPI 자동 갱신 시 필요)

- [ ] 한국은행 ECOS 오픈 API 인증키 발급 (https://ecos.bok.or.kr/)
  - 현재는 2000~2025년 데이터를 코드에 내장하여 사용 중
  - 연도별 갱신을 자동화할 경우 필요

### 결제 (Premium / Pro 구독)

- [ ] PG사 가맹점 등록 및 API Key 발급 (아래 중 선택)
  - 토스페이먼츠: 개발자 센터 가입 → 가맹점 등록
  - NHN KCP / 이니시스 / 카카오페이
- [ ] 사업자등록증 필수 (모든 PG 가맹에 요구)
- [ ] 정기 구독(빌링) API 연동 방식 결정 (자동결제 키 발급)

---

## 2. 인프라 / 서버

### 도메인 / SSL

- [ ] 도메인 구매 및 DNS 설정 (예: `memon.kr`)
- [ ] SSL 인증서 설정 (Let's Encrypt 또는 클라우드 제공 인증서)
- [ ] www → non-www 리다이렉트 설정 (또는 반대)

### 서버 호스팅

- [ ] 프론트엔드(31_webadmin) 호스팅 선택 및 설정
  - Vercel / Netlify / AWS CloudFront + S3 중 선택
  - 환경별 배포 브랜치 설정 (`develop` → 스테이징, `main` → 운영)
- [ ] 백엔드 API 서버 호스팅 (AWS EC2/ECS 또는 NHN Cloud)
- [ ] 데이터베이스 (PostgreSQL): AWS RDS 또는 NHN Cloud DB 인스턴스
- [ ] RabbitMQ: MQ 서버 설정 (이메일 발송 등 비동기 처리용)

### CDN / 성능

- [ ] 정적 리소스 CDN 설정 (이미지, JS, CSS)

---

## 3. 법적 / 규정

### 개인정보보호

- [ ] 개인정보처리방침 작성 및 서비스 내 게시
  - 수집 항목: 이름, 이메일, 경조사 기록 데이터
  - 수집 목적, 보유 기간, 제3자 제공 여부 명시
- [ ] 회원가입 시 개인정보 수집·이용 동의서 구현
- [ ] PIPA (개인정보보호법) 준수 여부 법률 검토
- [ ] Google AdSense 요구사항에 따른 쿠키 동의 배너 구현

### 사업자 등록

- [ ] 사업자등록증 발급 (PG 가맹 및 AdSense 수익 정산에 필수)
- [ ] 통신판매업 신고 (유료 구독 서비스 운영 시 필수)

### 이용약관

- [ ] 서비스 이용약관 작성 및 게시
- [ ] 유료 서비스 이용약관 (구독 플랜, 환불 정책) 작성
- [ ] 만 14세 미만 가입 제한 안내 문구 추가

---

## 4. 데이터 / 콘텐츠

### CPI 데이터 유지보수

- [ ] 매년 1월: 한국은행 CPI 확정치를 `tb_cpi_index` 테이블에 업데이트
- [ ] 매년 1월: 은행 평균 예금 이자율 업데이트 (물가환산 계산기 이자율 데이터)

### 스마트 금액 추천 기본값

- [ ] 경조사 유형별 글로벌 기본 금액 검증 (`tb_recommendation_default`)
  - 결혼 / 장례 / 생일 / 돌잔치 / 졸업 / 입학 등 유형별 초기 추천 금액 설정
  - 한국 경조사 시장 데이터를 기반으로 초기값 보정

---

## 5. 앱스토어 (32_appuser — 모바일 앱 출시 시)

### Google Play Store

- [ ] Google Play Console 개발자 계정 등록 (일회성 $25)
- [ ] 앱 등록 정보 준비 (스크린샷, 설명, 기능 그래픽, 앱 아이콘)
- [ ] 개인정보처리방침 URL 등록
- [ ] 콘텐츠 등급 설문 완료

### Apple App Store

- [ ] Apple Developer Program 등록 ($99/년)
- [ ] App Store Connect 앱 등록
- [ ] 앱 심사 가이드라인 준수 여부 검토
- [ ] 구독 상품 인앱 결제(IAP) 설정 (Apple 수수료 30% 반영한 가격 책정)

---

## 6. 마케팅 / 분석

### 분석 도구

- [ ] Google Analytics 4 (GA4) 설정 및 Measurement ID 발급
- [ ] Sentry 프로젝트 생성 및 DSN 발급 (에러 모니터링)

### SEO

- [ ] `sitemap.xml` 생성 및 웹 루트 배치
- [ ] `robots.txt` 설정
- [ ] Open Graph 메타태그 구현 (SNS 공유 미리보기)
- [ ] Google Search Console 사이트 등록 및 사이트맵 제출

---

## 7. 디자인 리소스

### 앱 아이콘 / 로고

- [ ] 앱 아이콘 디자인 (1024×1024 px, 다양한 플랫폼 사이즈 산출)
- [ ] 파비콘 생성 및 `public/favicon.ico` 교체 (현재 기본값)
- [ ] 로고 SVG 파일 제작

### 이메일 템플릿

- [ ] 관계연결 초대 이메일 HTML 템플릿
- [ ] 일정 / 생일 / 기념일 알림 이메일 HTML 템플릿
- [ ] 비밀번호 재설정 이메일 HTML 템플릿 (소셜 로그인 외 자체 가입 방식 지원 시)
