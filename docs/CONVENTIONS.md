# 프로젝트 컨벤션

이 문서는 Pigeon 프로젝트의 코드 및 협업 컨벤션을 정의합니다.

---

## 1. Git 컨벤션

### 1.1 브랜치 네이밍

| 브랜치 유형 | 네이밍 패턴                    | 설명                 |
| ----------- | ------------------------------ | -------------------- |
| 메인        | `main`                         | 프로덕션 배포 브랜치 |
| 개발        | `develop`                      | 개발 통합 브랜치     |
| 기능        | `feature/{기능명}-#{이슈번호}` | 새로운 기능 개발     |
| 버그 수정   | `fix/{버그명}-#{이슈번호}`     | 버그 수정            |
| 문서        | `docs/{문서명}-#{이슈번호}`    | 문서 작성/수정       |
| 릴리스      | `release-{버전}`               | 릴리스 준비          |
| 핫픽스      | `hotfix/{이슈명}-#{이슈번호}`  | 긴급 수정            |

> **Note**: 이슈 번호는 항상 브랜치명 맨 뒤에 `#`과 함께 위치합니다. GitHub 이슈와 연결하여 작업 추적이 용이합니다.

**예시:**

- `feature/email-classification-#42`
- `fix/folder-tree-rendering-#58`
- `docs/ui-spec-#1`
- `release-1.0.0`
- `hotfix/login-error-#99`

### 1.2 커밋 메시지 (Conventional Commits)

#### 기본 형식

```
<type>(<scope>): <subject>

<body>

<github-issue>
```

#### Type (필수)

| Type       | 설명                                           | 예시                       |
| ---------- | ---------------------------------------------- | -------------------------- |
| `feat`     | 새로운 기능 추가                               | 메일 분류 API 추가         |
| `fix`      | 버그 수정                                      | 폴더 트리 렌더링 오류 수정 |
| `docs`     | 문서 변경                                      | README 업데이트            |
| `style`    | 코드 포맷팅, 세미콜론 누락 등 (동작 변경 없음) | 들여쓰기 수정              |
| `refactor` | 코드 리팩토링 (기능 변경 없음)                 | 함수 분리                  |
| `test`     | 테스트 추가/수정                               | 분류 로직 단위 테스트 추가 |
| `chore`    | 빌드, 설정, 패키지 등 기타 변경                | 의존성 업데이트            |
| `perf`     | 성능 개선                                      | 쿼리 최적화                |
| `ci`       | CI/CD 설정 변경                                | GitHub Actions 추가        |

#### Scope (선택)

프로젝트 내 영향 받는 영역을 명시합니다.

| Scope            | 설명               |
| ---------------- | ------------------ |
| `api`            | 백엔드 API 관련    |
| `ui`             | 프론트엔드 UI 관련 |
| `classification` | 메일 분류 로직     |
| `folder`         | 폴더 관리 기능     |
| `db`             | 데이터베이스 관련  |
| `auth`           | 인증/권한 관련     |
| `config`         | 설정 파일 관련     |

#### Subject (필수) - 제목 규칙

- **50자 이내**로 작성
- **한글** 사용 (영어도 허용)
- 첫 글자 대문자 X, 마침표 X
- **명령형**으로 작성: "추가한다" (O), "추가했음" (X), "추가" (O)

#### Body (선택) - 본문 규칙

- 제목과 본문 사이 **빈 줄 1개** 필수
- **72자마다 줄바꿈** 권장
- **무엇을, 왜** 변경했는지 설명 (어떻게는 코드로)
- 글머리 기호(`-`) 사용 가능

#### GitHub 이슈 연동 (권장)

커밋과 GitHub 이슈를 연결하여 트래킹합니다.

| 형식             | 용도                                  | 예시            |
| ---------------- | ------------------------------------- | --------------- |
| `#{번호}`        | 이슈 참조                             | `#123`          |
| `Closes #{번호}` | 이슈 완료 처리 (PR 머지 시 자동 종료) | `Closes #123`   |
| `Fixes #{번호}`  | 버그 이슈 완료 처리                   | `Fixes #123`    |
| `Refs #{번호}`   | 관련 이슈 참조                        | `Refs #45, #67` |

> **Note**: GitHub에서는 `Closes`, `Fixes`, `Resolves` 키워드를 사용하면 PR 머지 시 해당 이슈가 자동으로 닫힙니다.

#### 커밋 메시지 예시

**간단한 커밋:**

```
feat(classification): LLM 기반 메일 분류 로직 추가

#42
```

**상세한 커밋:**

```
feat(classification): LLM 기반 메일 분류 로직 추가

- OpenAI API 연동 구현
- 폴더 경로 추천 알고리즘 구현
- 분류 결과 캐싱 적용

Closes #42
```

**버그 수정:**

```
fix(ui): 폴더 트리 렌더링 오류 수정

깊이 3 이상의 폴더가 표시되지 않는 문제 해결
재귀 렌더링 로직 수정

Fixes #58
```

**여러 이슈 관련:**

```
refactor(api): 메일 분류 API 응답 형식 변경

BREAKING CHANGE: 기존 `category` 필드가 `folderPath`로 변경됨

Refs #100, #101
Closes #99
```

#### 잘못된 커밋 메시지 예시

```
# Bad: type 없음
메일 분류 기능 추가

# Bad: 제목이 너무 김
feat(classification): 사용자가 받은 이메일을 LLM을 활용하여 자동으로 적절한 폴더에 분류하는 기능을 추가함

# Bad: 과거형 사용
feat(api): 메일 분류 API를 추가했습니다

# Bad: 마침표 사용
feat(api): 메일 분류 API 추가.

# Bad: GitHub 이슈 번호 없음 (트래킹 불가)
feat(api): 메일 분류 API 추가
```

---

## 2. 파일/디렉토리 네이밍

### 2.1 일반 규칙

| 대상               | 규칙                                 | 예시                                        |
| ------------------ | ------------------------------------ | ------------------------------------------- |
| 디렉토리           | kebab-case                           | `email-service`, `folder-tree`              |
| Python 파일        | snake_case                           | `email_classifier.py`, `folder_manager.py`  |
| TypeScript/JS 파일 | camelCase 또는 PascalCase (컴포넌트) | `emailService.ts`, `FolderTree.tsx`         |
| 테스트 파일        | `test_*.py` 또는 `*.test.ts`         | `test_classifier.py`, `FolderTree.test.tsx` |

### 2.2 문서 파일

| 대상      | 규칙                   | 예시                          |
| --------- | ---------------------- | ----------------------------- |
| 마크다운  | UPPER_SNAKE_CASE       | `README.md`, `CONVENTIONS.md` |
| 기능 문서 | `[기능명] 문서명` 형식 | `[메일분류] API명세.md`       |

---

## 3. 코드 스타일

### 3.1 Python

- PEP 8 준수
- 타입 힌트 사용 권장
- docstring: Google 스타일

### 3.2 TypeScript/JavaScript

- ESLint + Prettier 적용
- 함수형 컴포넌트 + Hooks 사용 (React)

---

## 4. PR 및 코드 리뷰

### 4.1 PR 제목

커밋 메시지와 동일한 Conventional Commits 형식 사용

### 4.2 PR 본문

```markdown
## Summary

- 변경 사항 요약

## Test Plan

- [ ] 테스트 항목 1
- [ ] 테스트 항목 2
```

---

_이 문서는 프로젝트 진행에 따라 업데이트됩니다._
