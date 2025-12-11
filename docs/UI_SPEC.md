# Pigeon UI 설계 문서

> **작성일**: 2025-12-10
> **버전**: v1.0
> **상태**: Draft
> **관련 이슈**: #1

---

## 1. 개요

Pigeon은 AI 기반 메일 분류 시스템으로, 사용자가 메일을 효율적으로 관리할 수 있는 직관적인 UI를 제공합니다.

### 1.1 디자인 원칙

- **심플함**: 필수 기능에 집중한 깔끔한 인터페이스
- **직관성**: 메일 클라이언트에 익숙한 3단 레이아웃
- **반응성**: 실시간 동기화 상태 및 분류 진행 표시

---

## 2. 화면 구성

### 2.1 전체 레이아웃

```
┌─────────────────────────────────────────────────────────────────┐
│  Header (로고, 사용자 정보, 동기화 버튼)                          │
├────────────┬─────────────────────┬──────────────────────────────┤
│            │                     │                              │
│  Sidebar   │     Mail List       │       Mail Detail            │
│  (폴더트리) │     (메일 목록)       │       (메일 상세)             │
│            │                     │                              │
│  240px     │      360px          │         flex-1               │
│            │                     │                              │
├────────────┴─────────────────────┴──────────────────────────────┤
│  StatusBar (동기화 상태, 메일 수, 마지막 확인 시간)                 │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 페이지 구조

| 경로         | 페이지      | 설명                       |
| ------------ | ----------- | -------------------------- |
| `/`          | 랜딩 페이지 | 서비스 소개 및 로그인 유도 |
| `/login`     | 로그인      | Gmail OAuth 연동           |
| `/callback`  | OAuth 콜백  | 인증 처리 후 리다이렉트    |
| `/mail`      | 메일함      | 메인 3단 레이아웃          |
| `/mail/[id]` | 메일 상세   | 특정 메일 상세 보기        |

---

## 3. 컴포넌트 설계

### 3.1 컴포넌트 계층 구조

```
app/
├── layout.tsx                 # 루트 레이아웃
├── page.tsx                   # 랜딩 페이지
│
├── (auth)/
│   ├── login/page.tsx         # 로그인 페이지
│   └── callback/page.tsx      # OAuth 콜백
│
└── (main)/
    ├── layout.tsx             # 3단 레이아웃
    └── mail/
        ├── page.tsx           # 메일 목록
        └── [id]/page.tsx      # 메일 상세

components/
├── ui/                        # 기본 UI 컴포넌트
│   ├── Button.tsx
│   ├── Input.tsx
│   ├── Modal.tsx
│   ├── Skeleton.tsx
│   ├── Badge.tsx
│   ├── ProgressBar.tsx
│   ├── Pagination.tsx
│   ├── SearchInput.tsx
│   ├── ContextMenu.tsx
│   └── Dropdown.tsx
│
├── layout/                    # 레이아웃 컴포넌트
│   ├── Header.tsx
│   ├── Sidebar.tsx
│   └── StatusBar.tsx
│
├── mail/                      # 메일 관련 컴포넌트
│   ├── MailList.tsx
│   ├── MailListItem.tsx
│   ├── MailDetail.tsx
│   ├── MailActions.tsx
│   ├── MailPagination.tsx
│   └── AttachmentList.tsx
│
├── folder/                    # 폴더 관련 컴포넌트
│   ├── FolderTree.tsx
│   ├── FolderTreeItem.tsx
│   ├── FolderBadge.tsx
│   ├── VirtualFolders.tsx     # 가상 폴더 (전체/안읽음/별표/미분류)
│   ├── FolderContextMenu.tsx
│   └── FolderMoveModal.tsx
│
└── sync/                      # 동기화 관련 컴포넌트
    ├── SyncButton.tsx
    ├── SyncProgress.tsx
    └── SyncStatusBar.tsx
```

### 3.2 주요 컴포넌트 명세

#### Header

```typescript
interface HeaderProps {
  user: User | null;
  searchQuery: string;
  onSearchChange: (query: string) => void;
  onSearchSubmit: () => void;
  onSync: () => void;
  onLogout: () => void;
  isSyncing: boolean;
}
```

#### VirtualFolders

```typescript
type VirtualFolderType = 'all' | 'unread' | 'starred' | 'unclassified';

interface VirtualFoldersProps {
  selectedType: VirtualFolderType | null;
  onSelect: (type: VirtualFolderType) => void;
  counts: {
    all: number;
    unread: number;
    starred: number;
    unclassified: number;
  };
}
```

#### FolderTree

```typescript
interface FolderTreeProps {
  folders: Folder[];
  selectedFolderId: number | null;
  onSelectFolder: (folderId: number) => void;
  onContextMenu: (e: MouseEvent, folderId: number) => void;
  onCreateFolder: () => void;
}
```

#### MailList

```typescript
interface MailListProps {
  mails: MailListItem[];
  selectedMailId: number | null;
  selectedMailIds: number[];
  pagination: Pagination;
  onSelectMail: (mailId: number) => void;
  onToggleSelect: (mailId: number) => void;
  onSelectAll: () => void;
  onPageChange: (page: number) => void;
  isLoading: boolean;
}
```

#### MailDetail

```typescript
interface MailDetailProps {
  mail: Mail | null;
  onMove: () => void;
  onToggleStar: () => void;
  onDelete: () => void;
  onDownloadAttachment: (attachmentId: string) => void;
}
```

#### AttachmentList

```typescript
interface AttachmentListProps {
  attachments: Attachment[];
  onDownload: (attachmentId: string) => void;
}
```

#### Pagination

```typescript
interface PaginationProps {
  page: number;
  pageSize: number;
  totalCount: number;
  totalPages: number;
  hasNext: boolean;
  hasPrev: boolean;
  onPageChange: (page: number) => void;
}
```

#### SyncProgress

```typescript
interface SyncProgressProps {
  status: SyncStatus;
  onStop: () => void;
}
```

#### FolderMoveModal

```typescript
interface FolderMoveModalProps {
  isOpen: boolean;
  folders: Folder[];
  onClose: () => void;
  onMove: (folderId: number) => void;
  onCreateFolder: (name: string, parentId?: number) => void;
}
```

---

## 4. 목업 데이터

### 4.1 사용자 (User)

```typescript
interface User {
  id: number;
  email: string;
  name: string;
  picture: string | null;
  is_initial_sync_done: boolean;
  last_sync_at: string | null;
}

// 목업 데이터
const mockUser: User = {
  id: 1,
  email: 'user@example.com',
  name: '홍길동',
  picture: 'https://via.placeholder.com/40',
  is_initial_sync_done: true,
  last_sync_at: '2025-12-10T10:00:00Z',
};
```

### 4.2 폴더 (Folder)

```typescript
interface Folder {
  id: number;
  name: string;
  path: string;
  depth: number;
  parent_id: number | null;
  mail_count: number;
  unread_count: number;
  order: number;
  children?: Folder[];
}

// 목업 데이터
const mockFolders: Folder[] = [
  {
    id: 1,
    name: '업무',
    path: '업무',
    depth: 0,
    parent_id: null,
    mail_count: 100,
    unread_count: 12,
    order: 0,
    children: [
      {
        id: 2,
        name: '프로젝트A',
        path: '업무/프로젝트A',
        depth: 1,
        parent_id: 1,
        mail_count: 50,
        unread_count: 5,
        order: 0,
        children: [],
      },
      {
        id: 3,
        name: '프로젝트B',
        path: '업무/프로젝트B',
        depth: 1,
        parent_id: 1,
        mail_count: 50,
        unread_count: 7,
        order: 1,
        children: [],
      },
    ],
  },
  {
    id: 4,
    name: '개인',
    path: '개인',
    depth: 0,
    parent_id: null,
    mail_count: 30,
    unread_count: 3,
    order: 1,
    children: [
      {
        id: 5,
        name: '쇼핑',
        path: '개인/쇼핑',
        depth: 1,
        parent_id: 4,
        mail_count: 20,
        unread_count: 3,
        order: 0,
        children: [],
      },
    ],
  },
  {
    id: 6,
    name: '뉴스레터',
    path: '뉴스레터',
    depth: 0,
    parent_id: null,
    mail_count: 15,
    unread_count: 8,
    order: 2,
    children: [],
  },
];
```

### 4.3 메일 (Mail)

```typescript
interface Recipient {
  type: 'to' | 'cc' | 'bcc';
  email: string;
  name: string;
}

interface Attachment {
  id: string;
  name: string;
  size: number;
  mime_type: string;
}

interface FolderSummary {
  id: number;
  name: string;
  path: string;
}

// 메일 목록용 (간략)
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
  received_at: string;
}

// 메일 상세용 (전체)
interface Mail extends MailListItem {
  recipients: Recipient[];
  body_html: string;
  attachments: Attachment[];
}

// 목업 데이터
const mockMails: Mail[] = [
  {
    id: 1,
    gmail_id: '18c1234567890abc',
    thread_id: '18c1234567890abc',
    subject: '[프로젝트A] 주간 회의록 공유드립니다',
    sender: '김팀장 <kim@company.com>',
    sender_email: 'kim@company.com',
    recipients: [{ type: 'to', name: '홍길동', email: 'user@example.com' }],
    snippet: '안녕하세요, 홍길동님. 이번 주 회의록을 공유드립니다...',
    body_html: `<p>안녕하세요, 홍길동님</p>
<p>이번 주 회의록을 공유드립니다.</p>
<h2>주요 논의 사항</h2>
<ol>
  <li>신규 기능 개발 일정 확정</li>
  <li>QA 테스트 계획 수립</li>
  <li>다음 스프린트 백로그 정리</li>
</ol>
<p>감사합니다.</p>`,
    folder: { id: 2, name: '프로젝트A', path: '업무/프로젝트A' },
    attachments: [
      { id: 'att-1', name: '회의록.pdf', size: 102400, mime_type: 'application/pdf' }
    ],
    has_attachments: true,
    is_read: false,
    is_starred: true,
    is_classified: true,
    received_at: '2025-12-10T09:30:00Z',
  },
  {
    id: 2,
    gmail_id: '18c1234567890abd',
    thread_id: '18c1234567890abd',
    subject: 'PR Review 요청: feat/email-classification',
    sender: 'GitHub <noreply@github.com>',
    sender_email: 'noreply@github.com',
    recipients: [{ type: 'to', name: '홍길동', email: 'user@example.com' }],
    snippet: '@developer님이 PR 리뷰를 요청했습니다...',
    body_html: `<p>@developer님이 PR 리뷰를 요청했습니다.</p>
<p><strong>Pull Request #42</strong><br>feat: LLM 기반 메일 분류 로직 구현</p>`,
    folder: { id: 3, name: '프로젝트B', path: '업무/프로젝트B' },
    attachments: [],
    has_attachments: false,
    is_read: false,
    is_starred: false,
    is_classified: true,
    received_at: '2025-12-10T08:15:00Z',
  },
  {
    id: 3,
    gmail_id: '18c1234567890abe',
    thread_id: '18c1234567890abe',
    subject: '주문하신 상품이 배송 완료되었습니다',
    sender: '쿠팡 <no-reply@coupang.com>',
    sender_email: 'no-reply@coupang.com',
    recipients: [{ type: 'to', name: '홍길동', email: 'user@example.com' }],
    snippet: '주문하신 상품이 배송 완료되었습니다...',
    body_html: `<p>홍길동님, 안녕하세요.</p>
<p>주문하신 상품이 배송 완료되었습니다.</p>
<p>주문번호: 2024121012345<br>상품명: 무선 키보드</p>`,
    folder: { id: 5, name: '쇼핑', path: '개인/쇼핑' },
    attachments: [],
    has_attachments: false,
    is_read: true,
    is_starred: false,
    is_classified: true,
    received_at: '2025-12-10T07:00:00Z',
  },
  {
    id: 4,
    gmail_id: '18c1234567890abf',
    thread_id: '18c1234567890abf',
    subject: 'TechNews Weekly #128 - AI 트렌드 총정리',
    sender: 'TechNews <newsletter@technews.com>',
    sender_email: 'newsletter@technews.com',
    recipients: [{ type: 'to', name: '홍길동', email: 'user@example.com' }],
    snippet: '이번 주 테크 뉴스 하이라이트...',
    body_html: `<h1>이번 주 테크 뉴스 하이라이트</h1>
<h2>AI 트렌드</h2>
<ul><li>Claude 4 출시 임박</li><li>GPT-5 루머 정리</li></ul>`,
    folder: { id: 6, name: '뉴스레터', path: '뉴스레터' },
    attachments: [],
    has_attachments: false,
    is_read: true,
    is_starred: false,
    is_classified: true,
    received_at: '2025-12-09T18:00:00Z',
  },
  {
    id: 5,
    gmail_id: '18c1234567890ac0',
    thread_id: '18c1234567890ac0',
    subject: '안녕하세요, 문의드립니다',
    sender: 'Unknown <unknown@example.org>',
    sender_email: 'unknown@example.org',
    recipients: [{ type: 'to', name: '홍길동', email: 'user@example.com' }],
    snippet: '안녕하세요. 일반적인 문의 메일입니다...',
    body_html: `<p>안녕하세요.</p><p>일반적인 문의 메일입니다.</p><p>감사합니다.</p>`,
    folder: null,
    attachments: [],
    has_attachments: false,
    is_read: false,
    is_starred: false,
    is_classified: false,
    received_at: '2025-12-09T15:30:00Z',
  },
];
```

### 4.4 동기화 상태 (SyncStatus)

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
  started_at: string | null;
  completed_at: string | null;
  estimated_remaining: number | null;  // seconds
}

// 목업 데이터
const mockSyncStatus: SyncStatus = {
  sync_id: 'sync_abc123',
  state: 'completed',
  type: 'incremental',
  progress: {
    total: 50,
    synced: 50,
    classified: 50,
    percentage: 100,
  },
  started_at: '2025-12-10T10:00:00Z',
  completed_at: '2025-12-10T10:02:00Z',
  estimated_remaining: null,
};

// 동기화 진행 중 예시
const mockSyncInProgress: SyncStatus = {
  sync_id: 'sync_xyz789',
  state: 'in_progress',
  type: 'initial',
  progress: {
    total: 500,
    synced: 120,
    classified: 100,
    percentage: 24,
  },
  started_at: '2025-12-10T11:00:00Z',
  completed_at: null,
  estimated_remaining: 180,
};
```

### 4.5 페이지네이션 (Pagination)

```typescript
interface Pagination {
  page: number;
  page_size: number;
  total_count: number;
  total_pages: number;
  has_next: boolean;
  has_prev: boolean;
}

// 목업 데이터
const mockPagination: Pagination = {
  page: 1,
  page_size: 20,
  total_count: 150,
  total_pages: 8,
  has_next: true,
  has_prev: false,
};
```

---

## 5. 디자인 시스템

### 5.1 색상 팔레트

```css
:root {
  /* Primary */
  --primary-50: #eff6ff;
  --primary-100: #dbeafe;
  --primary-500: #3b82f6;
  --primary-600: #2563eb;
  --primary-700: #1d4ed8;

  /* Gray */
  --gray-50: #f9fafb;
  --gray-100: #f3f4f6;
  --gray-200: #e5e7eb;
  --gray-300: #d1d5db;
  --gray-500: #6b7280;
  --gray-700: #374151;
  --gray-900: #111827;

  /* Semantic */
  --success: #10b981;
  --warning: #f59e0b;
  --error: #ef4444;
  --info: #3b82f6;

  /* Background */
  --bg-primary: #ffffff;
  --bg-secondary: #f9fafb;
  --bg-sidebar: #f3f4f6;
}
```

### 5.2 타이포그래피

```css
:root {
  --font-sans: 'Pretendard', -apple-system, BlinkMacSystemFont, system-ui, sans-serif;

  /* Font Sizes */
  --text-xs: 0.75rem; /* 12px */
  --text-sm: 0.875rem; /* 14px */
  --text-base: 1rem; /* 16px */
  --text-lg: 1.125rem; /* 18px */
  --text-xl: 1.25rem; /* 20px */
  --text-2xl: 1.5rem; /* 24px */
}
```

### 5.3 간격 (Spacing)

```css
:root {
  --space-1: 0.25rem; /* 4px */
  --space-2: 0.5rem; /* 8px */
  --space-3: 0.75rem; /* 12px */
  --space-4: 1rem; /* 16px */
  --space-5: 1.25rem; /* 20px */
  --space-6: 1.5rem; /* 24px */
  --space-8: 2rem; /* 32px */
}
```

### 5.4 컴포넌트 스타일

#### 버튼

```
Primary: bg-primary-600, text-white, hover:bg-primary-700
Secondary: bg-gray-100, text-gray-700, hover:bg-gray-200
Ghost: bg-transparent, text-gray-600, hover:bg-gray-100
Danger: bg-error, text-white, hover:opacity-90
```

#### 카드

```
bg-white, rounded-lg, shadow-sm, border border-gray-200
```

#### 입력 필드

```
border border-gray-300, rounded-md, focus:ring-2 focus:ring-primary-500
```

---

## 6. 상태 관리

### 6.1 Store 구조 (Zustand)

```typescript
// stores/authStore.ts
interface AuthStore {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: () => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
}

// stores/folderStore.ts
type VirtualFolderType = 'all' | 'unread' | 'starred' | 'unclassified' | null;

interface FolderStore {
  folders: Folder[];
  selectedFolderId: number | null;
  selectedVirtualFolder: VirtualFolderType;
  isLoading: boolean;

  // 조회
  fetchFolders: () => Promise<void>;
  selectFolder: (id: number) => void;
  selectVirtualFolder: (type: VirtualFolderType) => void;

  // 폴더 관리
  createFolder: (name: string, parentId?: number) => Promise<void>;
  updateFolder: (id: number, data: { name?: string; parentId?: number }) => Promise<void>;
  deleteFolder: (id: number) => Promise<void>;
  reorderFolders: (orders: { id: number; order: number }[]) => Promise<void>;
}

// stores/mailStore.ts
interface MailStore {
  mails: MailListItem[];
  selectedMail: Mail | null;
  selectedMailIds: number[];  // 다중 선택
  pagination: Pagination;
  searchQuery: string;
  isLoading: boolean;

  // 조회
  fetchMails: (params: {
    folderId?: number;
    isRead?: boolean;
    isStarred?: boolean;
    isClassified?: boolean;
    search?: string;
    page?: number;
  }) => Promise<void>;
  fetchMailDetail: (id: number) => Promise<void>;

  // 선택
  selectMail: (id: number) => void;
  toggleMailSelection: (id: number) => void;
  selectAllMails: () => void;
  clearSelection: () => void;

  // 상태 변경
  markAsRead: (ids: number[]) => Promise<void>;
  markAsUnread: (ids: number[]) => Promise<void>;
  toggleStar: (id: number) => Promise<void>;

  // 이동/삭제
  moveMails: (ids: number[], folderId: number) => Promise<void>;
  deleteMails: (ids: number[]) => Promise<void>;

  // 검색
  setSearchQuery: (query: string) => void;

  // 페이지네이션
  goToPage: (page: number) => void;
}

// stores/syncStore.ts
interface SyncStore {
  status: SyncStatus;
  isPolling: boolean;

  startSync: (fullSync?: boolean) => Promise<void>;
  stopSync: () => Promise<void>;
  pollStatus: () => Promise<void>;
  startPolling: () => void;
  stopPolling: () => void;
}

// stores/uiStore.ts (UI 상태)
interface UIStore {
  isSidebarOpen: boolean;
  isMoveModalOpen: boolean;
  isCreateFolderModalOpen: boolean;
  contextMenu: {
    isOpen: boolean;
    x: number;
    y: number;
    targetId: number | null;
    type: 'folder' | 'mail' | null;
  };

  toggleSidebar: () => void;
  openMoveModal: () => void;
  closeMoveModal: () => void;
  openContextMenu: (x: number, y: number, targetId: number, type: 'folder' | 'mail') => void;
  closeContextMenu: () => void;
}
```

---

## 7. 와이어프레임

### 7.1 랜딩 페이지 (`/`)

```
┌─────────────────────────────────────────────────────────────┐
│                         Header                               │
│  🕊️ Pigeon                              [Gmail로 시작하기]   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    🕊️                                       │
│              AI 메일 폴더링                                  │
│                                                             │
│     LLM이 당신의 메일을 자동으로 분류해드립니다               │
│                                                             │
│              [ Gmail로 시작하기 ]                            │
│                                                             │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                     │
│  │ 자동분류 │  │ 스마트   │  │ 실시간   │                     │
│  │ AI 기반  │  │ 폴더생성 │  │ 동기화   │                     │
│  └─────────┘  └─────────┘  └─────────┘                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 메일함 (`/mail`)

```
┌─────────────────────────────────────────────────────────────────────┐
│ 🕊️ Pigeon    [🔍 메일 검색...]     🔄 동기화  user@example.com ▼   │
├──────────────┬─────────────────────┬────────────────────────────────┤
│ 📁 폴더      │ 📬 메일목록         │ 📄 메일 상세                    │
│              │                     │                                │
│ ─ 가상 폴더 ─│ ☐ 전체선택  🗑️삭제  │ From: 김팀장 <kim@company.com> │
│ 📥 전체 (30) │ ─────────────────── │ To: 홍길동                     │
│ 📩 안읽음(15)│ ●⭐주간 회의록...   │ Date: 2025-12-10 09:30         │
│ ⭐ 별표 (3)  │   김팀장 | 09:30 📎 │ Subject: [프로젝트A] 주간...   │
│ 📂 미분류(2) │ ─────────────────── │ ─────────────────────────────  │
│ ───────────  │ ● PR Review 요청... │                                │
│ ─ 내 폴더 ── │   GitHub | 08:15   │ 안녕하세요, 홍길동님           │
│ ▼ 업무 (12) │ ─────────────────── │                                │
│   ├ 프로젝A │ ○ 배송 완료...      │ 이번 주 회의록을 공유...       │
│   └ 프로젝B │   쿠팡 | 07:00     │                                │
│ ▼ 개인 (3)  │ ─────────────────── │ ─────────────────────────────  │
│   └ 쇼핑    │                     │ 📎 첨부파일 (1)                │
│ ▶ 뉴스레터  │                     │  └ 📄 회의록.pdf (100KB) ⬇️    │
│  [+ 폴더]   │ ─────────────────── │ ─────────────────────────────  │
│             │ < 이전 1-20/150 다음>│ 📁 업무/프로젝트A              │
│             │                     │                                │
│             │                     │ [📁이동] [⭐별표] [🗑️삭제]     │
├──────────────┴─────────────────────┴────────────────────────────────┤
│ ✓ 동기화 완료 | 총 150개 메일 | 마지막 확인: 30초 전                │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.3 동기화 진행 중 메일함 (백그라운드 동기화)

동기화 중에도 메일함을 그대로 사용할 수 있습니다. 동기화된 메일은 실시간으로 목록에 추가됩니다.

```
┌─────────────────────────────────────────────────────────────────────┐
│ 🕊️ Pigeon    [🔍 메일 검색...]   🔄⟳ 동기화중  user@example.com ▼  │
├──────────────┬─────────────────────┬────────────────────────────────┤
│ 📁 폴더      │ 📬 메일목록         │ 📄 메일 상세                    │
│              │                     │                                │
│ ─ 가상 폴더 ─│ ☐ 전체선택  🗑️삭제  │  (메일을 선택하세요)            │
│ 📥 전체 (30) │ ─────────────────── │                                │
│ 📩 안읽음(15)│ ● 새로 도착한 메일  │                                │
│ ⭐ 별표 (3)  │   방금 동기화됨 ✨  │                                │
│ 📂 미분류(2) │ ─────────────────── │                                │
│ ───────────  │ ● 주간 회의록...    │                                │
│ ─ 내 폴더 ── │   김팀장 | 09:30 📎 │                                │
│ ▼ 업무 (12) │ ─────────────────── │                                │
│   ...        │ ...                 │                                │
│              │ ─────────────────── │                                │
│              │ < 이전 1-20/150 다음>│                                │
├──────────────┴─────────────────────┴────────────────────────────────┤
│ 🔄 동기화 중... 24% (120/500)  ← 클릭하여 상세 보기                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.4 동기화 상세 팝업 (상태바 클릭 시)

하단 상태바를 클릭하면 동기화 상세 정보가 팝업으로 표시됩니다.

```
                              ┌─────────────────────────────────┐
                              │  📥 동기화 진행 상황          ✕ │
                              ├─────────────────────────────────┤
                              │                                 │
                              │  ┌───────────────────────┐      │
                              │  │████████░░░░░░░░░░░░░░│ 24%  │
                              │  └───────────────────────┘      │
                              │                                 │
                              │  📬 메일 동기화: 120 / 500      │
                              │  🏷️ 메일 분류:   100 / 500      │
                              │                                 │
                              │  ⏱️ 예상 남은 시간: 약 3분       │
                              │                                 │
                              │          [동기화 중단]          │
                              │                                 │
                              └─────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────────┐
│ 🔄 동기화 중... 24% (120/500)  ← 클릭하여 상세 보기                  │
└─────────────────────────────────────────────────────────────────────┘
```

### 7.5 폴더 관리 (컨텍스트 메뉴)

```
폴더 우클릭 시:
┌────────────────┐
│ 📁 이름 변경   │
│ 📂 하위폴더 추가│
│ 🔢 순서 변경   │
│ ───────────── │
│ 🗑️ 폴더 삭제   │
└────────────────┘

폴더 이동 모달:
┌─────────────────────────────────┐
│  메일 이동                    ✕ │
├─────────────────────────────────┤
│  이동할 폴더를 선택하세요        │
│                                 │
│  ○ 📥 미분류                    │
│  ○ 📁 업무                      │
│    ○ 📁 프로젝트A               │
│    ○ 📁 프로젝트B               │
│  ○ 📁 개인                      │
│    ○ 📁 쇼핑                    │
│  ○ 📁 뉴스레터                  │
│                                 │
│  [+ 새 폴더 만들기]              │
│                                 │
│        [취소]  [이동]           │
└─────────────────────────────────┘
```

---

## 8. 반응형 설계

### 8.1 브레이크포인트

| 브레이크포인트 | 크기           | 레이아웃                    |
| -------------- | -------------- | --------------------------- |
| Mobile         | < 768px        | 단일 패널 (탭 전환)         |
| Tablet         | 768px - 1024px | 2단 (폴더 + 목록/상세 전환) |
| Desktop        | > 1024px       | 3단 전체 표시               |

### 8.2 모바일 레이아웃

```
┌───────────────────────┐
│ 🕊️ Pigeon    ☰  🔄   │
├───────────────────────┤
│ [폴더] [메일] [상세]   │  ← 탭 전환
├───────────────────────┤
│                       │
│  현재 선택된 뷰 표시   │
│                       │
└───────────────────────┘
```

---

## 9. 관련 문서

- [시스템 아키텍처](./ARCHITECTURE.md)
- [컨벤션 가이드](./CONVENTIONS.md)
- [기술 결정 기록](./DECISIONS.md)

---

_이 문서는 프로젝트 진행에 따라 지속적으로 업데이트됩니다._
