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