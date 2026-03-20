# GIT_POLICY.md

## 1. Subject 규칙

- 50자 이하
- 마침표 및 특수기호 금지
- 동사 원형 + 대문자 시작
- 완전한 문장 금지
- 한글로 작성

---

## 2. Commit 구조

type

body

footer

---

## 3. type 목록

- feat
- fix
- docs
- style
- refactor
- test
- chore

---

## 4. body

- 한 줄 72자 이내
- 무엇/왜 변경했는지 설명
- 한글로 작성

---

## 5. footer

- 유형: #이슈번호
- Fixes, Resolves, Ref, Related to
---

## 6. submodule

- submodule을 포함하고 있고, submodule의 변경 사항이 발생한 경우
  - submodule의 변경사항을 우선 commit
  - 최신화한 submodule의 commit hash를 포함하여 commit 진행

---

## 7. Claude Code 플러그인 활용

Git 작업 수행 시 다음 플러그인을 활용한다.

### 7.1 커밋 플러그인

| 플러그인 | 명령 | 용도 |
|---------|------|------|
| commit-commands | `/commit` | 구조화된 Git 커밋 생성 (섹션 1~5 규칙 자동 적용) |
| commit-commands | `/commit-push-pr` | 커밋 + 푸시 + PR 생성 일괄 수행 |
| commit-commands | `/clean_gone` | 원격에서 삭제된 로컬 브랜치 및 연관 worktree 정리 |

### 7.2 GitHub 플러그인

| 플러그인 | 명령 | 용도 |
|---------|------|------|
| github | `gh pr create` | PR 생성 |
| github | `gh pr comment` | PR 코멘트 작성 |
| github | `gh pr review` | PR 리뷰 승인/변경요청 |
| github | `gh issue` | 이슈 관리 |

### 7.3 Submodule 커밋 시 플러그인 순서

섹션 6의 submodule 규칙과 연동하여 다음 순서로 수행한다:

```
1. hs-common 변경 → /commit (hs-common 디렉토리)
2. webadmin 변경 → /commit (webadmin 디렉토리)
3. appuser 변경 → /commit (appuser 디렉토리)
4. workspace 해시 반영 → /commit (workspace 루트)
```

PR 생성이 필요한 경우, 마지막 단계에서 `/commit-push-pr`을 사용한다.