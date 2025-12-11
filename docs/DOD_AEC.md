# Pigeon DoD & AEC 정의서

> **작성일**: 2025-12-11
> **버전**: v1.0
> **목적**: Claude Code가 자율적으로 완료 여부를 판단하기 위한 기준 문서

---

## 1. 개요

이 문서는 각 작업(Story/Task)의 **완료 기준(DoD: Definition of Done)**과 **인수 조건(AEC: Acceptance Criteria)**을 정의합니다.

### 1.1 용어 정의

| 용어 | 설명 |
|------|------|
| **DoD (Definition of Done)** | 모든 Story에 공통으로 적용되는 완료 기준 |
| **AEC (Acceptance Criteria)** | 개별 Story가 충족해야 하는 구체적인 조건 |
| **Epic** | 큰 기능 단위 (GitHub Label로 관리) |
| **Story** | 사용자 관점의 기능 단위 (GitHub Issue로 관리) |
| **Task** | Story 내 구체적 작업 (Issue 내 체크리스트) |

### 1.2 GitHub Issues 구조

```
Epic (Label)
└── Story (Issue)
    ├── AEC (Issue 본문 체크리스트)
    └── Tasks (Issue 본문 체크리스트)
```

**Labels 체계**:
- `epic:auth`, `epic:folder`, `epic:mail`, `epic:sync`, `epic:classify`, `epic:ui`
- `phase-1`, `phase-2`, ..., `phase-8`
- `BE`, `FE`, `AI`
- `priority:critical`, `priority:high`, `priority:medium`

---

## 2. 전역 DoD (Global Definition of Done)

모든 Story는 아래 조건을 **모두** 충족해야 완료로 간주합니다.

### 2.1 코드 품질

- [ ] 린트 에러 없음 (Backend: Ruff, Frontend: ESLint)
- [ ] 타입 에러 없음 (Frontend: TypeScript strict mode)
- [ ] 콘솔 에러/경고 없음
- [ ] 하드코딩된 비밀값(API Key, Secret) 없음

### 2.2 컨벤션 준수 ([CONVENTIONS.md](./CONVENTIONS.md) 참조)

- [ ] **Git 컨벤션**
  - 브랜치명: `feature/{기능명}-#{이슈번호}`, `fix/{버그명}-#{이슈번호}` 형식
  - 커밋 메시지: Conventional Commits 형식 (`feat`, `fix`, `docs` 등)
  - GitHub Issue 연동: `Closes #번호` 또는 `Refs #번호` 포함
- [ ] **파일/디렉토리 네이밍**
  - Python: snake_case (`email_classifier.py`)
  - TypeScript/React: camelCase/PascalCase (`emailService.ts`, `FolderTree.tsx`)
  - 디렉토리: kebab-case (`email-service`)
- [ ] **코드 스타일**
  - Python: PEP 8 준수, 타입 힌트 사용
  - TypeScript: ESLint + Prettier 적용, 함수형 컴포넌트 + Hooks

### 2.3 기능 동작

- [ ] 해당 기능이 정상적으로 동작함
- [ ] 에러 케이스에서 적절한 에러 메시지 표시
- [ ] 로딩 상태가 적절히 표시됨 (UI의 경우)

### 2.4 API 규약 (Backend)

- [ ] API_SPEC.md에 정의된 Request/Response 형식 준수
- [ ] 적절한 HTTP 상태 코드 반환
- [ ] Swagger 문서 자동 생성됨

### 2.5 테스트

- [ ] 핵심 로직에 대한 단위 테스트 작성 (선택적)
- [ ] 수동 테스트로 정상 동작 확인

### 2.6 문서화

- [ ] 복잡한 로직에 주석 추가 (필요시)
- [ ] 새로운 환경변수 추가 시 README 또는 .env.example 업데이트

---

## 3. Phase별 Story & AEC

### Phase 1: 프로젝트 초기 설정

#### Epic: 프로젝트 기반 구축 (`epic:setup`)

---

##### Story 1.1: Django 프로젝트 초기화

**Labels**: `phase-1`, `BE`, `epic:setup`

**설명**: Django 프로젝트와 기본 앱 구조를 생성합니다.

**AEC (인수 조건)**:
- [ ] `backend/` 디렉토리에 Django 프로젝트 생성됨
- [ ] `python manage.py runserver` 실행 시 정상 구동
- [ ] 앱 구조 생성: `accounts`, `folders`, `mails`, `sync`, `classification`
- [ ] `config/settings/` 에 base.py, development.py, production.py 분리
- [ ] CORS 설정 완료 (django-cors-headers)
- [ ] drf-spectacular 설치 및 `/api/v1/docs/` 접속 가능

**Tasks**:
- [ ] uv로 프로젝트 초기화 (`uv init`)
- [ ] Django, DRF, drf-spectacular 설치
- [ ] 프로젝트 생성 (`django-admin startproject config .`)
- [ ] 앱 생성 (5개)
- [ ] settings 분리
- [ ] CORS, Swagger 설정

---

##### Story 1.2: Next.js 프로젝트 초기화

**Labels**: `phase-1`, `FE`, `epic:setup`

**설명**: Next.js 프로젝트와 기본 구조를 생성합니다.

**AEC (인수 조건)**:
- [ ] `frontend/` 디렉토리에 Next.js 14 프로젝트 생성됨 (App Router)
- [ ] `npm run dev` 실행 시 정상 구동
- [ ] TypeScript strict mode 활성화
- [ ] Tailwind CSS 설정 완료
- [ ] Zustand 설치 및 stores 디렉토리 구조 생성
- [ ] 기본 UI 컴포넌트 디렉토리 구조 생성 (`components/ui/`)

**Tasks**:
- [ ] pnpm으로 Next.js 프로젝트 생성
- [ ] Tailwind CSS 설정
- [ ] Zustand 설치
- [ ] 디렉토리 구조 생성 (components, stores, hooks, lib, types)
- [ ] ESLint, Prettier 설정

---

##### Story 1.3: 데이터베이스 모델 정의

**Labels**: `phase-1`, `BE`, `epic:setup`

**설명**: DATABASE.md 기반으로 Django 모델을 정의합니다.

**AEC (인수 조건)**:
- [ ] User 모델 정의 (AbstractUser 확장)
- [ ] Folder 모델 정의 (self-referential, depth, path)
- [ ] Mail 모델 정의 (JSONField for attachments, recipients)
- [ ] `python manage.py makemigrations` 성공
- [ ] `python manage.py migrate` 성공
- [ ] Django Admin에서 모델 확인 가능

**Tasks**:
- [ ] accounts/models.py - User 모델
- [ ] folders/models.py - Folder 모델
- [ ] mails/models.py - Mail 모델
- [ ] 마이그레이션 생성 및 실행
- [ ] Admin 등록

---

### Phase 2: 인증 시스템

#### Epic: Google OAuth2 인증 (`epic:auth`)

---

##### Story 2.1: Google OAuth2 로그인 API

**Labels**: `phase-2`, `BE`, `epic:auth`, `priority:critical`

**설명**: Google OAuth2를 통한 로그인 기능을 구현합니다.

**AEC (인수 조건)**:
- [ ] `GET /api/v1/auth/google/login/` 호출 시 Google OAuth URL로 리다이렉트
- [ ] `GET /api/v1/auth/google/callback/` 에서 authorization code를 토큰으로 교환
- [ ] 성공 시 JWT Access Token + Refresh Token 반환
- [ ] 신규 사용자인 경우 User 생성
- [ ] 기존 사용자인 경우 토큰만 발급
- [ ] Gmail API 접근을 위한 OAuth scope 포함 (`gmail.readonly`, `gmail.modify`)

**Tasks**:
- [ ] Google Cloud Console에서 OAuth 클라이언트 설정
- [ ] google-auth, google-api-python-client 설치
- [ ] 로그인 시작 View 구현
- [ ] 콜백 View 구현
- [ ] JWT 토큰 발급 로직 (djangorestframework-simplejwt)

---

##### Story 2.2: 토큰 갱신 및 로그아웃 API

**Labels**: `phase-2`, `BE`, `epic:auth`

**설명**: JWT 토큰 갱신과 로그아웃 기능을 구현합니다.

**AEC (인수 조건)**:
- [ ] `POST /api/v1/auth/token/refresh/` 호출 시 새 Access Token 발급
- [ ] Refresh Token 만료 시 `TOKEN_EXPIRED` 에러 반환
- [ ] `POST /api/v1/auth/logout/` 호출 시 토큰 무효화
- [ ] `GET /api/v1/users/me/` 호출 시 현재 사용자 정보 반환

**Tasks**:
- [ ] 토큰 갱신 View 구현
- [ ] 로그아웃 View 구현 (토큰 블랙리스트)
- [ ] 사용자 정보 View 구현
- [ ] JWT 인증 미들웨어 설정

---

##### Story 2.3: 프론트엔드 로그인 UI

**Labels**: `phase-2`, `FE`, `epic:auth`

**설명**: 로그인 페이지와 OAuth 콜백 처리를 구현합니다.

**AEC (인수 조건)**:
- [ ] `/` 랜딩 페이지에서 "Gmail로 시작하기" 버튼 표시
- [ ] 버튼 클릭 시 백엔드 OAuth URL로 리다이렉트
- [ ] `/callback` 페이지에서 토큰 수신 및 저장
- [ ] 로그인 성공 시 `/mail` 페이지로 이동
- [ ] Auth Store에 사용자 정보, 토큰 저장
- [ ] 토큰 만료 시 자동 갱신 로직

**Tasks**:
- [ ] 랜딩 페이지 (`app/page.tsx`)
- [ ] 로그인 페이지 (`app/(auth)/login/page.tsx`)
- [ ] 콜백 페이지 (`app/(auth)/callback/page.tsx`)
- [ ] Auth Store (`stores/authStore.ts`)
- [ ] API 클라이언트에 토큰 인터셉터 추가

---

### Phase 3: 핵심 API 개발

#### Epic: 폴더 관리 (`epic:folder`)

---

##### Story 3.1: 폴더 CRUD API

**Labels**: `phase-3`, `BE`, `epic:folder`

**설명**: 폴더 생성, 조회, 수정, 삭제 API를 구현합니다.

**AEC (인수 조건)**:
- [ ] `GET /api/v1/folders/` 호출 시 트리 구조로 폴더 목록 반환
- [ ] `GET /api/v1/folders/?flat=true` 호출 시 평탄화된 리스트 반환
- [ ] `POST /api/v1/folders/` 호출 시 새 폴더 생성
- [ ] 최대 5단계 깊이 제한 (초과 시 `MAX_DEPTH_EXCEEDED` 에러)
- [ ] `PATCH /api/v1/folders/{id}/` 호출 시 폴더 이름/위치 수정
- [ ] 순환 참조 방지 (자신 또는 하위로 이동 불가)
- [ ] `DELETE /api/v1/folders/{id}/` 호출 시 폴더 삭제
- [ ] 삭제된 폴더의 메일은 "미분류"로 이동
- [ ] `PUT /api/v1/folders/reorder/` 호출 시 순서 일괄 변경

**Tasks**:
- [ ] FolderSerializer 구현
- [ ] FolderViewSet 구현 (list, create, update, destroy)
- [ ] TreeManagerService 구현 (트리 구조 변환)
- [ ] 폴더 순서 변경 View 구현
- [ ] 유효성 검증 로직 (깊이, 순환 참조)

---

#### Epic: 메일 관리 (`epic:mail`)

---

##### Story 3.2: 메일 CRUD API

**Labels**: `phase-3`, `BE`, `epic:mail`

**설명**: 메일 조회, 상태 변경, 이동, 삭제 API를 구현합니다.

**AEC (인수 조건)**:
- [ ] `GET /api/v1/mails/` 호출 시 메일 목록 반환 (페이지네이션)
- [ ] `folder_id`, `is_read`, `is_starred`, `is_classified` 필터 동작
- [ ] `search` 파라미터로 제목/발신자 검색 가능
- [ ] `GET /api/v1/mails/{id}/` 호출 시 메일 상세 정보 반환
- [ ] 상세 조회 시 자동으로 `is_read = true` 처리
- [ ] `PATCH /api/v1/mails/{id}/` 호출 시 읽음/별표 상태 변경
- [ ] `POST /api/v1/mails/{id}/move/` 호출 시 폴더 이동
- [ ] `POST /api/v1/mails/bulk-move/` 호출 시 최대 100개 일괄 이동
- [ ] `POST /api/v1/mails/bulk-update/` 호출 시 일괄 상태 변경
- [ ] `DELETE /api/v1/mails/{id}/` 호출 시 Soft Delete

**Tasks**:
- [ ] MailSerializer 구현 (List용, Detail용 분리)
- [ ] MailViewSet 구현
- [ ] 필터링 로직 (django-filter 사용)
- [ ] 페이지네이션 (CursorPagination 또는 PageNumberPagination)
- [ ] Bulk 작업 View 구현

---

##### Story 3.3: 첨부파일 다운로드 API

**Labels**: `phase-3`, `BE`, `epic:mail`

**설명**: Gmail에서 첨부파일을 다운로드하는 API를 구현합니다.

**AEC (인수 조건)**:
- [ ] `GET /api/v1/mails/{id}/attachments/{att_id}/` 호출 시 파일 다운로드
- [ ] Gmail API를 통해 실시간으로 첨부파일 조회
- [ ] 적절한 Content-Type 헤더 설정
- [ ] Content-Disposition으로 파일명 전달
- [ ] 존재하지 않는 첨부파일 요청 시 `ATTACHMENT_NOT_FOUND` 에러

**Tasks**:
- [ ] GmailAPIClient에 첨부파일 다운로드 메서드 추가
- [ ] 첨부파일 다운로드 View 구현
- [ ] 스트리밍 응답 처리

---

### Phase 4: Gmail 동기화

#### Epic: Gmail 동기화 (`epic:sync`)

---

##### Story 4.1: Gmail API 클라이언트

**Labels**: `phase-4`, `BE`, `epic:sync`

**설명**: Gmail API와 통신하는 클라이언트를 구현합니다.

**AEC (인수 조건)**:
- [ ] OAuth2 토큰으로 Gmail API 인증
- [ ] `messages.list` API로 메일 목록 조회 가능
- [ ] `messages.get` API로 메일 상세 조회 가능
- [ ] 메일 본문 파싱 (HTML/Plain text)
- [ ] 첨부파일 메타데이터 추출
- [ ] Rate Limiting 적용 (429 에러 시 exponential backoff)

**Tasks**:
- [ ] GmailAPIClient 클래스 구현
- [ ] 메일 목록 조회 메서드
- [ ] 메일 상세 조회 메서드
- [ ] 본문 파싱 유틸리티
- [ ] Rate Limiter 구현

---

##### Story 4.2: 초기 동기화

**Labels**: `phase-4`, `BE`, `epic:sync`, `priority:critical`

**설명**: 최초 로그인 시 6개월치 메일을 동기화합니다.

**AEC (인수 조건)**:
- [ ] `POST /api/v1/sync/start/` 호출 시 동기화 시작
- [ ] 최근 6개월 메일만 조회 (after: 파라미터 사용)
- [ ] 20개 배치 단위로 메일 조회 및 저장
- [ ] 동기화 진행률 실시간 조회 가능 (`GET /api/v1/sync/status/`)
- [ ] 동기화 중단 가능 (`POST /api/v1/sync/stop/`)
- [ ] 동기화 완료 후 `is_initial_sync_done = true` 설정

**Tasks**:
- [ ] SyncStatus 모델 (또는 캐시) 구현
- [ ] 동기화 시작 View
- [ ] 동기화 상태 조회 View
- [ ] 동기화 중단 View
- [ ] 배치 동기화 로직 (GmailSyncService)

---

##### Story 4.3: 증분 동기화

**Labels**: `phase-4`, `BE`, `epic:sync`

**설명**: 새로운 메일을 주기적으로 감지하고 동기화합니다.

**AEC (인수 조건)**:
- [ ] Gmail History API를 사용하여 새 메일 감지
- [ ] 마지막 동기화 이후의 변경사항만 조회
- [ ] 새 메일 발견 시 즉시 DB에 저장
- [ ] 3분 주기로 백그라운드 폴링 (또는 프론트엔드 요청)

**Tasks**:
- [ ] historyId 기반 증분 동기화 로직
- [ ] 폴링 엔드포인트 또는 스케줄러 구현
- [ ] 새 메일 감지 시 분류 서비스 연동

---

### Phase 5: AI 분류 시스템

#### Epic: AI 메일 분류 (`epic:classify`)

---

##### Story 5.1: LLM 클라이언트 설정

**Labels**: `phase-5`, `AI`, `epic:classify`

**설명**: LangChain + Gemini API 연동을 설정합니다.

**AEC (인수 조건)**:
- [ ] LangChain 설치 및 Gemini 연동 완료
- [ ] `GOOGLE_API_KEY` 환경변수로 인증
- [ ] 테스트 프롬프트 호출 시 정상 응답
- [ ] Rate Limiting 적용 (15 RPM 무료 tier)

**Tasks**:
- [ ] langchain, langchain-google-genai 설치
- [ ] LLMClient 클래스 구현
- [ ] 연결 테스트

---

##### Story 5.2: 분류 프롬프트 설계

**Labels**: `phase-5`, `AI`, `epic:classify`

**설명**: 메일 분류를 위한 프롬프트를 설계합니다.

**AEC (인수 조건)**:
- [ ] 메일 정보(제목, 발신자, 본문 요약)를 입력으로 받음
- [ ] 기존 폴더 목록을 컨텍스트로 전달
- [ ] JSON 형식으로 분류 결과 반환 (폴더 경로, 신규 여부, 분류 이유)
- [ ] 프롬프트 템플릿 파일로 관리 (`prompts.py`)

**Tasks**:
- [ ] 분류 프롬프트 템플릿 작성
- [ ] PromptBuilder 클래스 구현
- [ ] 응답 파싱 로직 (ResponseParser)

---

##### Story 5.3: 분류 서비스 구현

**Labels**: `phase-5`, `AI`, `epic:classify`, `priority:critical`

**설명**: 메일을 분류하고 폴더에 배정하는 서비스를 구현합니다.

**AEC (인수 조건)**:
- [ ] 메일 ID 리스트를 받아 일괄 분류
- [ ] 기존 폴더와 매칭되면 해당 폴더로 배정
- [ ] 새 폴더가 필요하면 자동 생성 (최대 5단계)
- [ ] 분류 실패 시 1회 재시도 후 "미분류" 폴더로 이동
- [ ] 분류 결과를 Mail 모델에 저장 (`folder_id`, `is_classified = true`)
- [ ] `POST /api/v1/classification/classify/` API 동작
- [ ] `GET /api/v1/classification/{id}/` API 동작
- [ ] `POST /api/v1/classification/classify-unclassified/` API 동작

**Tasks**:
- [ ] ClassifierService 구현
- [ ] 폴더 매칭/생성 로직
- [ ] 에러 핸들링 (재시도, 미분류 처리)
- [ ] 분류 API View 구현
- [ ] 동기화 시 자동 분류 연동

---

### Phase 6: Frontend 개발

#### Epic: UI 구현 (`epic:ui`)

---

##### Story 6.1: 3단 레이아웃 구현

**Labels**: `phase-6`, `FE`, `epic:ui`

**설명**: 폴더 트리 | 메일 목록 | 메일 상세 3단 레이아웃을 구현합니다.

**AEC (인수 조건)**:
- [ ] 좌측: 폴더 트리 (250px 고정)
- [ ] 중앙: 메일 목록 (유동 너비)
- [ ] 우측: 메일 상세 (400px 고정, 선택 시 표시)
- [ ] 반응형 고려 (모바일에서는 단일 컬럼)
- [ ] Header: 로고, 사용자 정보, 동기화 버튼
- [ ] StatusBar: 동기화 상태, 메일 통계

**Tasks**:
- [ ] 메인 레이아웃 (`app/(main)/layout.tsx`)
- [ ] Header 컴포넌트
- [ ] Sidebar 컴포넌트 (폴더 영역)
- [ ] StatusBar 컴포넌트

---

##### Story 6.2: 폴더 트리 컴포넌트

**Labels**: `phase-6`, `FE`, `epic:ui`

**설명**: 다계층 폴더 트리 UI를 구현합니다.

**AEC (인수 조건)**:
- [ ] 가상 폴더 표시 (전체, 읽지 않음, 별표, 미분류)
- [ ] 실제 폴더 트리 표시 (접기/펼치기)
- [ ] 폴더별 메일 개수, 안읽음 개수 표시
- [ ] 폴더 클릭 시 해당 폴더 메일 목록 표시
- [ ] 폴더 우클릭 시 컨텍스트 메뉴 (이름 변경, 삭제)
- [ ] 새 폴더 추가 버튼

**Tasks**:
- [ ] FolderTree 컴포넌트
- [ ] FolderTreeItem 컴포넌트 (재귀)
- [ ] VirtualFolders 컴포넌트
- [ ] FolderContextMenu 컴포넌트
- [ ] Folder Store 연동 (`stores/folderStore.ts`)

---

##### Story 6.3: 메일 목록 컴포넌트

**Labels**: `phase-6`, `FE`, `epic:ui`

**설명**: 메일 목록과 페이지네이션을 구현합니다.

**AEC (인수 조건)**:
- [ ] 메일 목록 표시 (발신자, 제목, 시간, 미리보기)
- [ ] 읽음/안읽음 시각적 구분 (볼드체 등)
- [ ] 별표, 첨부파일 아이콘 표시
- [ ] 메일 클릭 시 상세 보기
- [ ] 체크박스로 다중 선택
- [ ] 일괄 작업 버튼 (삭제, 읽음 처리, 이동)
- [ ] 페이지네이션 (이전/다음)
- [ ] 로딩 상태 스켈레톤 UI

**Tasks**:
- [ ] MailList 컴포넌트
- [ ] MailListItem 컴포넌트
- [ ] MailPagination 컴포넌트
- [ ] MailBulkActions 컴포넌트
- [ ] Mail Store 연동 (`stores/mailStore.ts`)

---

##### Story 6.4: 메일 상세 컴포넌트

**Labels**: `phase-6`, `FE`, `epic:ui`

**설명**: 메일 상세 내용을 표시합니다.

**AEC (인수 조건)**:
- [ ] 발신자, 수신자, 제목, 날짜 표시
- [ ] HTML 본문 렌더링 (sanitize 처리)
- [ ] 첨부파일 목록 표시 및 다운로드
- [ ] AI 분류 정보 표시 (폴더 경로, 분류 이유)
- [ ] 액션 버튼 (폴더 이동, 별표, 삭제)
- [ ] 메일 선택 안됨 시 빈 상태 표시

**Tasks**:
- [ ] MailDetail 컴포넌트
- [ ] AttachmentList 컴포넌트
- [ ] MailActions 컴포넌트
- [ ] ClassificationInfo 컴포넌트

---

##### Story 6.5: 동기화 UI

**Labels**: `phase-6`, `FE`, `epic:ui`

**설명**: 동기화 상태를 표시하고 제어합니다.

**AEC (인수 조건)**:
- [ ] Header에 동기화 버튼 (새로고침 아이콘)
- [ ] 동기화 중 회전 애니메이션
- [ ] StatusBar에 진행률 표시 (초기 동기화 시)
- [ ] 프로그레스 바 또는 퍼센티지 표시
- [ ] 동기화 완료/실패 토스트 알림
- [ ] 마지막 동기화 시간 표시

**Tasks**:
- [ ] SyncButton 컴포넌트
- [ ] SyncProgress 컴포넌트
- [ ] Sync Store 연동 (`stores/syncStore.ts`)
- [ ] Toast 알림 시스템

---

### Phase 7: 통합 및 테스트

#### Epic: QA (`epic:qa`)

---

##### Story 7.1: E2E 플로우 테스트

**Labels**: `phase-7`, `ALL`, `epic:qa`

**설명**: 전체 플로우를 수동/자동으로 테스트합니다.

**AEC (인수 조건)**:
- [ ] 로그인 → 초기 동기화 → 메일 목록 표시 플로우 정상
- [ ] 폴더 선택 → 메일 필터링 정상
- [ ] 메일 상세 보기 정상
- [ ] 메일 이동, 삭제, 읽음 처리 정상
- [ ] 새 메일 감지 (증분 동기화) 정상
- [ ] 에러 발생 시 적절한 에러 메시지 표시

**Tasks**:
- [ ] 수동 테스트 체크리스트 작성
- [ ] Playwright E2E 테스트 작성 (선택적)
- [ ] 버그 발견 시 Issue 생성

---

##### Story 7.2: 버그 수정 및 최적화

**Labels**: `phase-7`, `ALL`, `epic:qa`

**설명**: 발견된 버그를 수정하고 성능을 최적화합니다.

**AEC (인수 조건)**:
- [ ] 콘솔 에러 없음
- [ ] 네트워크 에러 시 재시도 로직 동작
- [ ] 불필요한 API 호출 제거
- [ ] 메모리 누수 없음

**Tasks**:
- [ ] 버그 수정
- [ ] 성능 프로파일링
- [ ] 코드 정리

---

### Phase 8: 배포

#### Epic: 배포 (`epic:deploy`)

---

##### Story 8.1: Backend 배포 (Railway)

**Labels**: `phase-8`, `BE`, `epic:deploy`

**설명**: Django 백엔드를 Railway에 배포합니다.

**AEC (인수 조건)**:
- [ ] Railway 프로젝트 생성
- [ ] 환경변수 설정 (SECRET_KEY, DATABASE_URL, GOOGLE_* 등)
- [ ] Gunicorn으로 프로덕션 서버 실행
- [ ] HTTPS 적용
- [ ] API 엔드포인트 접속 가능
- [ ] Swagger UI 접속 가능

**Tasks**:
- [ ] railway.json 또는 Procfile 작성
- [ ] requirements.txt 또는 pyproject.toml 확인
- [ ] 환경변수 설정
- [ ] 배포 및 테스트

---

##### Story 8.2: Frontend 배포 (Vercel)

**Labels**: `phase-8`, `FE`, `epic:deploy`

**설명**: Next.js 프론트엔드를 Vercel에 배포합니다.

**AEC (인수 조건)**:
- [ ] Vercel 프로젝트 연결
- [ ] 환경변수 설정 (NEXT_PUBLIC_API_URL)
- [ ] 빌드 성공
- [ ] 배포 URL 접속 가능
- [ ] API 연동 정상 동작

**Tasks**:
- [ ] Vercel 프로젝트 설정
- [ ] 환경변수 설정
- [ ] 배포 및 테스트

---

##### Story 8.3: 프로덕션 최종 테스트

**Labels**: `phase-8`, `ALL`, `epic:deploy`

**설명**: 프로덕션 환경에서 전체 플로우를 테스트합니다.

**AEC (인수 조건)**:
- [ ] 프로덕션 URL로 로그인 가능
- [ ] OAuth 리다이렉트 정상 (프로덕션 콜백 URL)
- [ ] 메일 동기화 정상
- [ ] 메일 분류 정상
- [ ] 전체 기능 정상 동작

**Tasks**:
- [ ] Google Cloud Console에 프로덕션 리다이렉트 URI 추가
- [ ] 전체 플로우 수동 테스트
- [ ] 최종 확인

---

## 4. GitHub Issue 템플릿

### 4.1 Story Issue 템플릿

```markdown
## 설명
[Story에 대한 간단한 설명]

## AEC (인수 조건)
- [ ] 조건 1
- [ ] 조건 2
- [ ] 조건 3

## Tasks
- [ ] Task 1
- [ ] Task 2
- [ ] Task 3

## 참고 문서
- [API_SPEC.md](./API_SPEC.md)
- [DATABASE.md](./DATABASE.md)

## DoD 체크리스트
- [ ] 린트 에러 없음
- [ ] 타입 에러 없음
- [ ] 기능 정상 동작
- [ ] API 규약 준수 (Backend의 경우)
```

---

## 5. 관련 문서

- [작업 계획서](./WORK_PLAN.md)
- [API 명세서](./API_SPEC.md)
- [데이터베이스 설계](./DATABASE.md)
- [UI 설계](./UI_SPEC.md)
- [시스템 아키텍처](./ARCHITECTURE.md)

---

*이 문서는 프로젝트 진행에 따라 지속적으로 업데이트됩니다.*
