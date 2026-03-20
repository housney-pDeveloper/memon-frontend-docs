# 코드 리뷰 체크리스트

> 프론트엔드 프로젝트(webadmin, appuser, hs-common) 전체에 적용되는 코드 리뷰 체크리스트.

---

## 1. FSD 아키텍처 (webadmin)

- [ ] 레이어 의존성 방향 준수 (`app → widgets → features → entities → shared`)
- [ ] entities 간 교차 참조 없음 (entities/auth → entities/notification 금지)
- [ ] features 간 교차 참조 없음
- [ ] shared에서 entities/features/widgets 참조 없음
- [ ] barrel import 미사용 (`@/features/auth` 금지, `@/features/auth/model/useLogin` 사용)
- [ ] entities: read-only(GET만), features: write/actions(POST/PUT/PATCH/DELETE)
- [ ] primitives는 shared 내에서만 import

## 2. Light FSD (appuser)

- [ ] core/ 레이어를 통한 플랫폼 API 접근 (직접 localStorage/window 접근 금지)
- [ ] Radix UI 미사용 (순수 Tailwind CSS만)
- [ ] v2(React Native) 전환 시 재사용 가능한 구조

## 3. 코딩 컨벤션

- [ ] 컴포넌트: PascalCase.tsx, Arrow Function, displayName 있음
- [ ] Props 인터페이스 분리 선언 (`{ComponentName}Props`)
- [ ] `buildApiUrl()` 사용 (직접 URL 문자열 금지)
- [ ] `getApiErrorMessage()` / `parseApiError()` 사용 (`@hs/common/api`)
- [ ] 에러 메시지: 서버 응답 그대로 (FE 매핑 없음)
- [ ] 상수: `UPPER_SNAKE_CASE`
- [ ] `any` 타입 미사용
- [ ] `console.log` 미포함

## 4. API 규칙

- [ ] entities API: GET 요청만 (`entities/{domain}/api/`)
- [ ] features API: POST/PUT/PATCH/DELETE (`features/{domain}/api/`)
- [ ] 응답 타입: `ServerResponse<T>` 사용
- [ ] 에러 유틸: `@hs/common/api`에서 import

## 5. 상태 관리

- [ ] 서버 상태: TanStack Query 사용 (Zustand/useState로 API 데이터 관리 금지)
- [ ] 클라이언트 상태: Zustand 사용
- [ ] 폼 상태: React Hook Form + Zod
- [ ] 쿼리 키 일관성: `['domain', 'type', params]`
- [ ] 뮤테이션 성공 시 적절한 쿼리 무효화 (`invalidateQueries`)
- [ ] Zustand selector 사용 (전체 스토어 구독 방지)

## 6. 보안

- [ ] Access Token: 메모리 전용 (스토리지 저장 금지)
- [ ] `dangerouslySetInnerHTML` 미사용 (불가피 시 sanitize)
- [ ] 민감 정보(토큰, 비밀번호) 콘솔 로그 미출력
- [ ] `.env` 파일에 시크릿 키 직접 포함 안 함

## 7. 성능

- [ ] 불필요한 리렌더링 없음 (Zustand selector, memo 적절 사용)
- [ ] 대용량 리스트 가상화 또는 페이지네이션 적용
- [ ] 이미지 최적화 (Next.js Image 또는 lazy loading)
- [ ] 검색/필터 시 debounce 적용

## 8. 타입 안전성

- [ ] `any` 미사용
- [ ] 적절한 타입 좁힘 (type narrowing)
- [ ] API 응답 타입 정의 (`ServerResponse<T>`)
- [ ] Props 타입 정확성

## 9. hs-common 변경 시 추가 체크

- [ ] 외부 패키지 의존성 없음 (순수 TypeScript)
- [ ] 플랫폼 중립 (브라우저 전용 API 미사용)
- [ ] 하위 호환성 유지 (기존 시그니처 변경 시 양쪽 확인)
- [ ] webadmin 빌드 통과
- [ ] appuser 빌드 통과

---

## 심각도 기준

| 심각도 | 기준 | 조치 |
|--------|------|------|
| Critical | 버그, 보안 취약점, 데이터 손실 | 반드시 수정 후 재리뷰 |
| Major | 아키텍처 위반, 성능 문제 | 수정 권장 |
| Minor | 컨벤션 미준수, 가독성 | 수정 권장 (필수 아님) |
| Suggestion | 개선 아이디어 | 참고용 |

---

## Claude Code 플러그인 활용

코드 리뷰 수행 시 다음 플러그인을 활용한다.

| 플러그인 | 명령 | 활용 시점 | 용도 |
|---------|------|----------|------|
| code-review | `/code-review` | PR 기반 리뷰 시 | 체계적 코드 리뷰 수행, PR diff 분석 |
| simplify | `/simplify` | 리뷰 완료 후 | 코드 재사용·품질·효율성 개선 점검 |
| github | `gh` CLI | 리뷰 결과 공유 시 | PR 코멘트 작성, 리뷰 상태 관리 |

### 활용 프로세스

1. PR이 존재하는 경우 → `/code-review`로 위 체크리스트 기반 리뷰 수행
2. 리뷰 완료 후 → `/simplify`로 변경된 코드의 품질·재사용·효율성 개선 점검
3. 리뷰 결과를 GitHub PR에 코멘트로 기록 (`gh pr comment`, `gh pr review`)
4. Team 수행 시 → 리뷰 결과를 result_VERIFICATION.md에 기록

### 04_qa-team 수행 시 플러그인 순서

```
code-reviewer 수행 (게이트)
    │ → `/code-review` + `/simplify` 활용
    ▼ Critical/Major 0건 시에만 진행
security-auditor 수행
    │ → `/security-guidance` 활용
    ▼
종합 판정 → result_VERIFICATION.md
```
