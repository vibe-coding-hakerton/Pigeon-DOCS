# Pigeon API 명세서

> **작성일**: 2025-12-10
> **버전**: v1.0
> **상태**: Draft
> **문서화 도구**: Swagger (drf-spectacular)

---

## 1. 개요

### 1.1 기본 정보

| 항목 | 값 |
|------|-----|
| Base URL (개발) | `http://localhost:8000/api/v1` |
| Base URL (운영) | `https://api.pigeon.railway.app/api/v1` |
| API 버전 | v1 |
| 인증 방식 | JWT (Access Token + Refresh Token) |
| 콘텐츠 타입 | `application/json` |

### 1.2 Swagger UI 접속

- 개발: `http://localhost:8000/api/v1/docs/`
- 스키마: `http://localhost:8000/api/v1/schema/`

### 1.3 공통 응답 형식

#### 성공 응답
```json
{
  "status": "success",
  "data": { ... }
}
```

#### 에러 응답
```json
{
  "status": "error",
  "code": "ERROR_CODE",
  "message": "에러 메시지",
  "details": { ... }
}
```

### 1.4 공통 에러 코드

| HTTP 상태 | 코드 | 설명 |
|-----------|------|------|
| 400 | `INVALID_REQUEST` | 잘못된 요청 |
| 401 | `UNAUTHORIZED` | 인증 필요 |
| 401 | `TOKEN_EXPIRED` | 토큰 만료 |
| 403 | `FORBIDDEN` | 권한 없음 |
| 404 | `NOT_FOUND` | 리소스 없음 |
| 429 | `RATE_LIMITED` | 요청 제한 초과 |
| 500 | `INTERNAL_ERROR` | 서버 에러 |

---

## 2. 인증 API (Auth)

### 2.1 Google OAuth 로그인 시작

Google OAuth2 인증 URL을 반환합니다.

```
GET /api/v1/auth/google/login/
```

#### Response (302 Redirect)
Google 로그인 페이지로 리다이렉트

---

### 2.2 Google OAuth 콜백

Google 로그인 완료 후 콜백 처리. JWT 토큰 발급.

```
GET /api/v1/auth/google/callback/
```

#### Query Parameters

| 이름 | 타입 | 필수 | 설명 |
|------|------|------|------|
| code | string | O | Google OAuth 인증 코드 |
| state | string | O | CSRF 방지용 state |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600,
    "user": {
      "id": 1,
      "email": "user@gmail.com",
      "name": "홍길동",
      "picture": "https://lh3.googleusercontent.com/..."
    }
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `OAUTH_FAILED` | OAuth 인증 실패 |
| `INVALID_STATE` | state 값 불일치 |

---

### 2.3 토큰 갱신

Access Token 만료 시 Refresh Token으로 갱신합니다.

```
POST /api/v1/auth/token/refresh/
```

#### Request Body
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIs..."
}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "expires_in": 3600
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `INVALID_REFRESH_TOKEN` | Refresh Token 유효하지 않음 |
| `TOKEN_EXPIRED` | Refresh Token 만료 |

---

### 2.4 로그아웃

현재 세션 로그아웃 (토큰 무효화)

```
POST /api/v1/auth/logout/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "message": "로그아웃 되었습니다."
}
```

---

## 3. 사용자 API (Users)

### 3.1 내 정보 조회

현재 로그인한 사용자 정보를 조회합니다.

```
GET /api/v1/users/me/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "email": "user@gmail.com",
    "name": "홍길동",
    "picture": "https://lh3.googleusercontent.com/...",
    "is_initial_sync_done": true,
    "last_sync_at": "2025-12-10T10:30:00Z",
    "created_at": "2025-12-10T09:00:00Z"
  }
}
```

---

## 4. 폴더 API (Folders)

### 4.1 폴더 트리 조회

사용자의 모든 폴더를 트리 구조로 조회합니다.

```
GET /api/v1/folders/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Query Parameters

| 이름 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| flat | boolean | X | false | true이면 평탄화된 리스트로 반환 |

#### Response (200 OK) - 트리 구조
```json
{
  "status": "success",
  "data": {
    "folders": [
      {
        "id": 1,
        "name": "업무",
        "path": "업무",
        "depth": 0,
        "mail_count": 150,
        "unread_count": 5,
        "order": 0,
        "children": [
          {
            "id": 2,
            "name": "프로젝트A",
            "path": "업무/프로젝트A",
            "depth": 1,
            "mail_count": 50,
            "unread_count": 2,
            "order": 0,
            "children": []
          }
        ]
      },
      {
        "id": 3,
        "name": "개인",
        "path": "개인",
        "depth": 0,
        "mail_count": 80,
        "unread_count": 10,
        "order": 1,
        "children": []
      }
    ],
    "total_mail_count": 230,
    "total_unread_count": 15
  }
}
```

#### Response (200 OK) - 평탄화 리스트 (flat=true)
```json
{
  "status": "success",
  "data": {
    "folders": [
      {
        "id": 1,
        "name": "업무",
        "path": "업무",
        "depth": 0,
        "parent_id": null,
        "mail_count": 150,
        "unread_count": 5
      },
      {
        "id": 2,
        "name": "프로젝트A",
        "path": "업무/프로젝트A",
        "depth": 1,
        "parent_id": 1,
        "mail_count": 50,
        "unread_count": 2
      }
    ]
  }
}
```

---

### 4.2 폴더 생성

새 폴더를 생성합니다. 최대 5단계 깊이까지 허용.

```
POST /api/v1/folders/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "name": "새 폴더",
  "parent_id": 1
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | string | O | 폴더 이름 (1-100자) |
| parent_id | integer | X | 상위 폴더 ID (null이면 루트) |

#### Response (201 Created)
```json
{
  "status": "success",
  "data": {
    "id": 4,
    "name": "새 폴더",
    "path": "업무/새 폴더",
    "depth": 1,
    "parent_id": 1,
    "mail_count": 0,
    "unread_count": 0,
    "order": 2,
    "created_at": "2025-12-10T11:00:00Z"
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `MAX_DEPTH_EXCEEDED` | 최대 깊이(5단계) 초과 |
| `DUPLICATE_FOLDER_PATH` | 동일 경로 폴더 존재 |
| `PARENT_NOT_FOUND` | 상위 폴더 없음 |

---

### 4.3 폴더 수정

폴더 이름 또는 위치를 수정합니다.

```
PATCH /api/v1/folders/{folder_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Path Parameters

| 이름 | 타입 | 설명 |
|------|------|------|
| folder_id | integer | 폴더 ID |

#### Request Body
```json
{
  "name": "수정된 이름",
  "parent_id": 2,
  "order": 0
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| name | string | X | 새 폴더 이름 |
| parent_id | integer | X | 새 상위 폴더 ID |
| order | integer | X | 정렬 순서 |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 4,
    "name": "수정된 이름",
    "path": "업무/프로젝트A/수정된 이름",
    "depth": 2,
    "parent_id": 2,
    "mail_count": 0,
    "unread_count": 0,
    "order": 0,
    "updated_at": "2025-12-10T11:30:00Z"
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `FOLDER_NOT_FOUND` | 폴더 없음 |
| `CIRCULAR_REFERENCE` | 순환 참조 (자기 자신 또는 하위로 이동) |
| `MAX_DEPTH_EXCEEDED` | 이동 후 최대 깊이 초과 |

---

### 4.4 폴더 삭제

폴더를 삭제합니다. 하위 폴더와 메일은 "미분류"로 이동.

```
DELETE /api/v1/folders/{folder_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Path Parameters

| 이름 | 타입 | 설명 |
|------|------|------|
| folder_id | integer | 폴더 ID |

#### Response (200 OK)
```json
{
  "status": "success",
  "message": "폴더가 삭제되었습니다.",
  "data": {
    "moved_mails_count": 25,
    "moved_subfolders_count": 2
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `FOLDER_NOT_FOUND` | 폴더 없음 |

---

### 4.5 폴더 순서 변경 (일괄)

여러 폴더의 순서를 한번에 변경합니다.

```
PUT /api/v1/folders/reorder/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "orders": [
    { "id": 1, "order": 0 },
    { "id": 3, "order": 1 },
    { "id": 2, "order": 2 }
  ]
}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "message": "폴더 순서가 변경되었습니다."
}
```

---

## 5. 메일 API (Mails)

### 5.1 메일 목록 조회

폴더별/전체 메일 목록을 조회합니다.

```
GET /api/v1/mails/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Query Parameters

| 이름 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| folder_id | integer | X | - | 폴더 ID (없으면 전체) |
| is_read | boolean | X | - | 읽음 상태 필터 |
| is_starred | boolean | X | - | 별표 상태 필터 |
| is_classified | boolean | X | - | 분류 상태 필터 |
| search | string | X | - | 검색어 (제목, 발신자) |
| page | integer | X | 1 | 페이지 번호 |
| page_size | integer | X | 20 | 페이지당 항목 수 (max: 100) |
| ordering | string | X | -received_at | 정렬 기준 |

**정렬 옵션 (ordering)**:
- `received_at`: 수신일 오름차순
- `-received_at`: 수신일 내림차순 (기본값)
- `sender`: 발신자 오름차순
- `subject`: 제목 오름차순

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "mails": [
      {
        "id": 1,
        "gmail_id": "18c1234567890abc",
        "thread_id": "18c1234567890abc",
        "subject": "12월 프로젝트 회의 안건",
        "sender": "김철수 <kim@company.com>",
        "sender_email": "kim@company.com",
        "snippet": "안녕하세요, 다음 주 회의 안건을 공유드립니다...",
        "folder": {
          "id": 2,
          "name": "프로젝트A",
          "path": "업무/프로젝트A"
        },
        "has_attachments": true,
        "is_read": false,
        "is_starred": false,
        "is_classified": true,
        "received_at": "2025-12-10T09:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "page_size": 20,
      "total_count": 150,
      "total_pages": 8,
      "has_next": true,
      "has_prev": false
    }
  }
}
```

---

### 5.2 메일 상세 조회

메일 전체 내용을 조회합니다.

```
GET /api/v1/mails/{mail_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Path Parameters

| 이름 | 타입 | 설명 |
|------|------|------|
| mail_id | integer | 메일 ID |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "gmail_id": "18c1234567890abc",
    "thread_id": "18c1234567890abc",
    "subject": "12월 프로젝트 회의 안건",
    "sender": "김철수 <kim@company.com>",
    "sender_email": "kim@company.com",
    "recipients": [
      { "type": "to", "email": "user@gmail.com", "name": "홍길동" },
      { "type": "cc", "email": "team@company.com", "name": "팀원" }
    ],
    "snippet": "안녕하세요, 다음 주 회의 안건을 공유드립니다...",
    "body_html": "<html><body>안녕하세요...</body></html>",
    "folder": {
      "id": 2,
      "name": "프로젝트A",
      "path": "업무/프로젝트A"
    },
    "attachments": [
      {
        "id": "ANGjdJ8xxx",
        "name": "회의안건.pdf",
        "size": 102400,
        "mime_type": "application/pdf"
      }
    ],
    "has_attachments": true,
    "is_read": true,
    "is_starred": false,
    "is_classified": true,
    "received_at": "2025-12-10T09:30:00Z",
    "created_at": "2025-12-10T09:35:00Z"
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `MAIL_NOT_FOUND` | 메일 없음 |

---

### 5.3 메일 상태 수정

메일의 읽음/별표 상태를 변경합니다.

```
PATCH /api/v1/mails/{mail_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "is_read": true,
  "is_starred": true
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| is_read | boolean | X | 읽음 상태 |
| is_starred | boolean | X | 별표 상태 |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "is_read": true,
    "is_starred": true,
    "updated_at": "2025-12-10T11:00:00Z"
  }
}
```

---

### 5.4 메일 폴더 이동

메일을 다른 폴더로 이동합니다.

```
POST /api/v1/mails/{mail_id}/move/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "folder_id": 3
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| folder_id | integer | O | 이동할 폴더 ID (null이면 미분류) |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "folder": {
      "id": 3,
      "name": "개인",
      "path": "개인"
    },
    "updated_at": "2025-12-10T11:00:00Z"
  }
}
```

#### Error Response
| 코드 | 설명 |
|------|------|
| `MAIL_NOT_FOUND` | 메일 없음 |
| `FOLDER_NOT_FOUND` | 폴더 없음 |

---

### 5.5 메일 일괄 폴더 이동

여러 메일을 한번에 다른 폴더로 이동합니다.

```
POST /api/v1/mails/bulk-move/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "mail_ids": [1, 2, 3, 4, 5],
  "folder_id": 3
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| mail_ids | array[integer] | O | 메일 ID 목록 (max: 100) |
| folder_id | integer | O | 이동할 폴더 ID |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "moved_count": 5,
    "folder": {
      "id": 3,
      "name": "개인",
      "path": "개인"
    }
  }
}
```

---

### 5.6 메일 일괄 상태 변경

여러 메일의 읽음/별표 상태를 한번에 변경합니다.

```
POST /api/v1/mails/bulk-update/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "mail_ids": [1, 2, 3],
  "is_read": true
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| mail_ids | array[integer] | O | 메일 ID 목록 (max: 100) |
| is_read | boolean | X | 읽음 상태 |
| is_starred | boolean | X | 별표 상태 |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "updated_count": 3
  }
}
```

---

### 5.7 메일 삭제 (Soft Delete)

메일을 삭제합니다 (Soft Delete).

```
DELETE /api/v1/mails/{mail_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "message": "메일이 삭제되었습니다."
}
```

---

### 5.8 첨부파일 다운로드

메일의 첨부파일을 다운로드합니다.

```
GET /api/v1/mails/{mail_id}/attachments/{attachment_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Path Parameters

| 이름 | 타입 | 설명 |
|------|------|------|
| mail_id | integer | 메일 ID |
| attachment_id | string | 첨부파일 ID (Gmail attachment ID) |

#### Response (200 OK)
- Content-Type: 첨부파일의 MIME Type
- Content-Disposition: `attachment; filename="파일명.pdf"`
- Body: 파일 바이너리 데이터

#### Error Response
| 코드 | 설명 |
|------|------|
| `MAIL_NOT_FOUND` | 메일 없음 |
| `ATTACHMENT_NOT_FOUND` | 첨부파일 없음 |
| `GMAIL_API_ERROR` | Gmail API 호출 실패 |

---

## 6. 동기화 API (Sync)

### 6.1 동기화 시작

Gmail 메일 동기화를 시작합니다.

```
POST /api/v1/sync/start/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Request Body (선택)
```json
{
  "full_sync": false
}
```

| 필드 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| full_sync | boolean | X | false | true이면 전체 재동기화 |

#### Response (202 Accepted)
```json
{
  "status": "success",
  "data": {
    "sync_id": "sync_abc123",
    "type": "incremental",
    "started_at": "2025-12-10T11:00:00Z"
  }
}
```

---

### 6.2 동기화 상태 조회

현재 동기화 진행 상태를 조회합니다.

```
GET /api/v1/sync/status/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (200 OK) - 동기화 중
```json
{
  "status": "success",
  "data": {
    "sync_id": "sync_abc123",
    "state": "in_progress",
    "type": "initial",
    "progress": {
      "total": 500,
      "synced": 120,
      "classified": 100,
      "percentage": 24
    },
    "started_at": "2025-12-10T11:00:00Z",
    "estimated_remaining": 180
  }
}
```

#### Response (200 OK) - 동기화 완료
```json
{
  "status": "success",
  "data": {
    "sync_id": "sync_abc123",
    "state": "completed",
    "type": "incremental",
    "progress": {
      "total": 15,
      "synced": 15,
      "classified": 15,
      "percentage": 100
    },
    "started_at": "2025-12-10T11:00:00Z",
    "completed_at": "2025-12-10T11:02:00Z"
  }
}
```

#### 동기화 상태 (state)

| 값 | 설명 |
|-----|------|
| `idle` | 동기화 대기 중 |
| `in_progress` | 동기화 진행 중 |
| `completed` | 동기화 완료 |
| `failed` | 동기화 실패 |

#### 동기화 유형 (type)

| 값 | 설명 |
|-----|------|
| `initial` | 최초 동기화 (6개월) |
| `incremental` | 증분 동기화 (새 메일만) |

---

### 6.3 동기화 중단

진행 중인 동기화를 중단합니다.

```
POST /api/v1/sync/stop/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (200 OK)
```json
{
  "status": "success",
  "message": "동기화가 중단되었습니다.",
  "data": {
    "sync_id": "sync_abc123",
    "synced_count": 120
  }
}
```

---

## 7. 분류 API (Classification)

### 7.1 메일 분류 요청

특정 메일의 분류를 요청합니다 (수동 재분류).

```
POST /api/v1/classification/classify/
```

#### Headers
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

#### Request Body
```json
{
  "mail_ids": [1, 2, 3]
}
```

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| mail_ids | array[integer] | O | 분류할 메일 ID 목록 (max: 20) |

#### Response (202 Accepted)
```json
{
  "status": "success",
  "data": {
    "classification_id": "cls_xyz789",
    "mail_count": 3,
    "started_at": "2025-12-10T11:00:00Z"
  }
}
```

---

### 7.2 분류 결과 조회

분류 작업 결과를 조회합니다.

```
GET /api/v1/classification/{classification_id}/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Path Parameters

| 이름 | 타입 | 설명 |
|------|------|------|
| classification_id | string | 분류 작업 ID |

#### Response (200 OK)
```json
{
  "status": "success",
  "data": {
    "classification_id": "cls_xyz789",
    "state": "completed",
    "results": [
      {
        "mail_id": 1,
        "status": "success",
        "folder": {
          "id": 2,
          "name": "프로젝트A",
          "path": "업무/프로젝트A"
        },
        "is_new_folder": false,
        "confidence": 0.95
      },
      {
        "mail_id": 2,
        "status": "success",
        "folder": {
          "id": 5,
          "name": "뉴스레터",
          "path": "뉴스레터"
        },
        "is_new_folder": true,
        "confidence": 0.88
      },
      {
        "mail_id": 3,
        "status": "failed",
        "error": "LLM_API_ERROR",
        "folder": null
      }
    ],
    "summary": {
      "total": 3,
      "success": 2,
      "failed": 1,
      "new_folders_created": 1
    },
    "started_at": "2025-12-10T11:00:00Z",
    "completed_at": "2025-12-10T11:00:05Z"
  }
}
```

#### 분류 상태 (state)

| 값 | 설명 |
|-----|------|
| `pending` | 대기 중 |
| `in_progress` | 진행 중 |
| `completed` | 완료 |
| `failed` | 실패 |

---

### 7.3 미분류 메일 일괄 분류

미분류 상태인 모든 메일을 분류합니다.

```
POST /api/v1/classification/classify-unclassified/
```

#### Headers
```
Authorization: Bearer {access_token}
```

#### Response (202 Accepted)
```json
{
  "status": "success",
  "data": {
    "classification_id": "cls_bulk123",
    "mail_count": 45,
    "started_at": "2025-12-10T11:00:00Z"
  }
}
```

---

## 8. 가상 폴더 API (Virtual Folders)

실제 폴더가 아닌 필터 기반 가상 폴더를 조회합니다.

### 8.1 읽지 않은 메일

```
GET /api/v1/mails/?is_read=false
```

### 8.2 별표 메일

```
GET /api/v1/mails/?is_starred=true
```

### 8.3 미분류 메일

```
GET /api/v1/mails/?is_classified=false
```

### 8.4 전체 메일

```
GET /api/v1/mails/
```

---

## 9. 데이터 타입 정의

### 9.1 User

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  picture: string | null;
  is_initial_sync_done: boolean;
  last_sync_at: string | null;  // ISO 8601
  created_at: string;  // ISO 8601
}
```

### 9.2 Folder

```typescript
interface Folder {
  id: number;
  name: string;
  path: string;
  depth: number;  // 0-4
  parent_id: number | null;
  mail_count: number;
  unread_count: number;
  order: number;
  created_at: string;  // ISO 8601
  updated_at: string;  // ISO 8601
}

interface FolderTree extends Folder {
  children: FolderTree[];
}
```

### 9.3 Mail

```typescript
interface Mail {
  id: number;
  gmail_id: string;
  thread_id: string;
  subject: string;
  sender: string;
  sender_email: string;
  recipients: Recipient[];
  snippet: string;
  body_html: string;
  folder: FolderSummary | null;
  attachments: Attachment[];
  has_attachments: boolean;
  is_read: boolean;
  is_starred: boolean;
  is_classified: boolean;
  received_at: string;  // ISO 8601
  created_at: string;  // ISO 8601
  updated_at: string;  // ISO 8601
}

interface MailListItem {
  id: number;
  gmail_id: string;
  thread_id: string;
  subject: string;
  sender: string;
  sender_email: string;
  snippet: string;
  folder: FolderSummary | null;
  has_attachments: boolean;
  is_read: boolean;
  is_starred: boolean;
  is_classified: boolean;
  received_at: string;  // ISO 8601
}

interface FolderSummary {
  id: number;
  name: string;
  path: string;
}

interface Recipient {
  type: 'to' | 'cc' | 'bcc';
  email: string;
  name: string;
}

interface Attachment {
  id: string;  // Gmail attachment ID
  name: string;
  size: number;  // bytes
  mime_type: string;
}
```

### 9.4 Pagination

```typescript
interface Pagination {
  page: number;
  page_size: number;
  total_count: number;
  total_pages: number;
  has_next: boolean;
  has_prev: boolean;
}
```

### 9.5 SyncStatus

```typescript
interface SyncStatus {
  sync_id: string;
  state: 'idle' | 'in_progress' | 'completed' | 'failed';
  type: 'initial' | 'incremental';
  progress: {
    total: number;
    synced: number;
    classified: number;
    percentage: number;
  };
  started_at: string | null;  // ISO 8601
  completed_at: string | null;  // ISO 8601
  estimated_remaining: number | null;  // seconds
}
```

---

## 10. API 엔드포인트 요약

| 카테고리 | Method | Endpoint | 설명 |
|----------|--------|----------|------|
| **인증** | GET | `/api/v1/auth/google/login/` | Google 로그인 시작 |
| | GET | `/api/v1/auth/google/callback/` | OAuth 콜백 |
| | POST | `/api/v1/auth/token/refresh/` | 토큰 갱신 |
| | POST | `/api/v1/auth/logout/` | 로그아웃 |
| **사용자** | GET | `/api/v1/users/me/` | 내 정보 조회 |
| **폴더** | GET | `/api/v1/folders/` | 폴더 트리 조회 |
| | POST | `/api/v1/folders/` | 폴더 생성 |
| | PATCH | `/api/v1/folders/{id}/` | 폴더 수정 |
| | DELETE | `/api/v1/folders/{id}/` | 폴더 삭제 |
| | PUT | `/api/v1/folders/reorder/` | 폴더 순서 변경 |
| **메일** | GET | `/api/v1/mails/` | 메일 목록 조회 |
| | GET | `/api/v1/mails/{id}/` | 메일 상세 조회 |
| | PATCH | `/api/v1/mails/{id}/` | 메일 상태 수정 |
| | POST | `/api/v1/mails/{id}/move/` | 메일 폴더 이동 |
| | POST | `/api/v1/mails/bulk-move/` | 메일 일괄 이동 |
| | POST | `/api/v1/mails/bulk-update/` | 메일 일괄 상태 변경 |
| | DELETE | `/api/v1/mails/{id}/` | 메일 삭제 |
| | GET | `/api/v1/mails/{id}/attachments/{att_id}/` | 첨부파일 다운로드 |
| **동기화** | POST | `/api/v1/sync/start/` | 동기화 시작 |
| | GET | `/api/v1/sync/status/` | 동기화 상태 조회 |
| | POST | `/api/v1/sync/stop/` | 동기화 중단 |
| **분류** | POST | `/api/v1/classification/classify/` | 메일 분류 요청 |
| | GET | `/api/v1/classification/{id}/` | 분류 결과 조회 |
| | POST | `/api/v1/classification/classify-unclassified/` | 미분류 일괄 분류 |

---

## 11. 관련 문서

- [데이터베이스 설계](./DATABASE.md)
- [시스템 아키텍처](./ARCHITECTURE.md)
- [기술 결정 기록](./DECISIONS.md)

---

*이 문서는 프로젝트 진행에 따라 지속적으로 업데이트됩니다.*
