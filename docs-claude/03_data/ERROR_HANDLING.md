# 에러 처리 가이드

## 1. 에러 코드 체계 (5자리)

### 대분류

| 대분류 | 코드 범위 | 설명 |
|--------|-----------|------|
| Gateway | 1XXXX | JWT 인증, API 정책, 라우팅, 서버 오류 |
| Application | 3XXXX | 비즈니스 로직 오류 |
| Common | 9XXXX | 공통 시스템 오류 |

### 세부 코드 범위

| 코드 범위 | 설명 | 비고 |
|-----------|------|------|
| 11XXX | JWT 인증 오류 | 11001~11006: 자동 리프레시/리다이렉트 |
| 12XXX | API 버전 정책 오류 | |
| 13XXX | 라우팅 오류 | |
| 19XXX | 서버 오류 | |
| 31XXX | 계정/인증 관련 | |
| 32XXX | 보안 관련 | |
| 33XXX | 데이터 관련 | |
| 90XXX | 시스템 공통 | |
| 91XXX | 요청/파라미터 | |
| 92XXX | 인증/인가 | 92001: 자동 리다이렉트 |
| 93XXX | 리소스 | |

## 2. 핵심 원칙: 서버 메시지 그대로 표시

- 프론트엔드에서 에러 코드별 메시지를 매핑하지 않는다.
- 서버 응답의 `message` 필드를 사용자에게 그대로 표시한다.
- `getApiErrorMessage(error, defaultMessage)`를 사용한다.
- `defaultMessage`는 서버 메시지가 없을 때만 사용된다.

## 3. 자동 처리되는 에러

다음 에러들은 Axios 응답 인터셉터에서 자동으로 처리된다.

| 에러 유형 | 코드 | 처리 방식 |
|-----------|------|-----------|
| JWT 에러 | 11001~11006 | 토큰 리프레시 → 실패 시 로그인 리다이렉트 |
| 인증 오류 | 92001 | 토큰 리프레시 → 실패 시 로그인 리다이렉트 |
| 403 Forbidden | - | `/error/403` 페이지 리다이렉트 |
| 네트워크 에러 | `NETWORK_ERROR` | "네트워크 오류가 발생했습니다." 메시지 |

## 4. 수동 처리 패턴

### 방법 1: onError 콜백

```typescript
import { getApiErrorMessage } from '@hs/common/api'

useMutation({
  mutationFn: createItem,
  onError: (error) => {
    toast.error(getApiErrorMessage(error, '생성에 실패했습니다.'))
  }
})
```

### 방법 2: try-catch

```typescript
import { getApiErrorMessage } from '@hs/common/api'

try {
  await mutation.mutateAsync(payload)
  toast.success('생성되었습니다.')
} catch (error) {
  toast.error(getApiErrorMessage(error, '생성에 실패했습니다.'))
}
```

## 5. ErrorBoundary

- `shared/components/ErrorBoundary.tsx`에서 전역 에러 경계를 제공한다.
- React 렌더링 에러를 포착한다.
- 사용자에게 에러 화면을 표시하고 재시도 옵션을 제공한다.
