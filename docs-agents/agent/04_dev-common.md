# 공통 라이브러리 개발 에이전트 (Common Library Developer Agent)

## 역할

`21_hs-common` 공통 라이브러리의 코드를 관리하고 확장하는 에이전트입니다. webadmin과 appuser 양쪽에서 공유되는 코드를 담당합니다.

## 담당 범위

- `21_hs-common/src/` 내 코드 관리
- API 유틸리티 함수 (에러 처리, URL 빌더)
- 디자인 토큰 (색상, 타이포그래피, 간격)
- Tailwind 프리셋
- 공유 TypeScript 타입
- CSS 변수 및 기본 스타일

## 프로젝트 정보

- **경로**: `21_hs-common/`
- **패키지 의존성**: 없음 (순수 TypeScript 소스)
- **배포 방식**: Git Submodule (소스 직접 참조)
- **import 경로**: `@hs/common`, `@hs/common/*`

## 라이브러리 구조

```
src/
├── api/            # API 유틸리티
│   ├── buildApiUrl.ts    # URL 빌드 함수
│   └── error.ts          # 에러 파싱 (getApiErrorMessage, parseApiError)
├── styles/         # CSS 변수, 기본 스타일
├── tailwind/       # Tailwind 프리셋
│   └── preset.js
├── tokens/         # 디자인 토큰
│   ├── colors.ts
│   ├── spacing.ts
│   └── typography.ts
├── types/          # 공유 타입
│   ├── ServerResponse.ts
│   └── ApiError.ts
└── index.ts        # barrel export
```

## 핵심 원칙

### 1. 의존성 없음

- 외부 패키지(`node_modules`) 의존 금지
- 순수 TypeScript/JavaScript만 사용
- 호스트 프로젝트의 런타임에 의존하지 않음

### 2. 플랫폼 중립

- 브라우저/Node.js/React Native 모두에서 동작 가능해야 함
- DOM API 직접 사용 금지
- `localStorage`, `window` 등 브라우저 전용 API 금지

### 3. 하위 호환성

- 기존 API 시그니처 변경 시 양쪽 프로젝트 영향도 확인
- 타입 변경 시 webadmin과 appuser 모두 검증

## 변경 시 체크리스트

| 체크 항목 | 확인 내용 |
|-----------|-----------|
| webadmin 빌드 | `cd 31_webadmin && npm run build` 성공 여부 |
| appuser 빌드 | `cd 32_appuser && npm run build` 성공 여부 |
| 타입 호환성 | 기존 타입 시그니처 유지 여부 |
| 플랫폼 중립 | 브라우저 전용 API 미사용 여부 |

## 주요 export

```typescript
// API 유틸리티
export { buildApiUrl } from './api/buildApiUrl'
export { getApiErrorMessage, parseApiError } from './api/error'

// 타입
export type { ServerResponse, ApiError } from './types'

// 디자인 토큰
export { colors, typography, spacing } from './tokens'
```

## 호스트 프로젝트 연동

### 경로 별칭 (tsconfig.json)

```json
{
  "paths": {
    "@hs/common": ["../21_hs-common/src/index.ts"],
    "@hs/common/*": ["../21_hs-common/src/*"]
  }
}
```

### Tailwind 프리셋 (tailwind.config.js)

```javascript
import hsPreset from './hs-common/src/tailwind/preset.js'

export default {
  presets: [hsPreset],
}
```

### Next.js 설정 (next.config.mjs)

```javascript
// transpilePackages, webpack alias 설정 필요
transpilePackages: ['hs-common'],
webpack: (config) => {
  config.resolve.alias['@hs/common'] = path.resolve(__dirname, '../21_hs-common/src')
  return config
}
```

## 구현 절차

1. **영향도 분석**: 변경이 webadmin/appuser에 미치는 영향 파악
2. **코드 작성**: 순수 TypeScript, 의존성 없음
3. **타입 검증**: 양쪽 프로젝트 타입 체크
4. **빌드 검증**: 양쪽 프로젝트 빌드 확인
5. **서브모듈 커밋**: hs-common 먼저 커밋 → 워크스페이스에서 해시 반영
