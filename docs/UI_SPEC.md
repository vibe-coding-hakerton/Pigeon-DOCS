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
│   └── Badge.tsx
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
│   └── MailActions.tsx
│
└── folder/                    # 폴더 관련 컴포넌트
    ├── FolderTree.tsx
    ├── FolderTreeItem.tsx
    └── FolderBadge.tsx
```

### 3.2 주요 컴포넌트 명세

#### Header

```typescript
interface HeaderProps {
  user: User | null;
  onSync: () => void;
  onLogout: () => void;
  isSyncing: boolean;
}
```

#### FolderTree

```typescript
interface FolderTreeProps {
  folders: Folder[];
  selectedFolderId: string | null;
  onSelectFolder: (folderId: string) => void;
}
```

#### MailList

```typescript
interface MailListProps {
  mails: Mail[];
  selectedMailId: string | null;
  onSelectMail: (mailId: string) => void;
  isLoading: boolean;
}
```

#### MailDetail

```typescript
interface MailDetailProps {
  mail: Mail | null;
  onMove: (folderId: string) => void;
  onDelete: () => void;
}
```

---

## 4. 목업 데이터

### 4.1 사용자 (User)

```typescript
interface User {
  id: string;
  email: string;
  name: string;
  profileImage?: string;
}

// 목업 데이터
const mockUser: User = {
  id: 'user-1',
  email: 'user@example.com',
  name: '홍길동',
  profileImage: 'https://via.placeholder.com/40',
};
```

### 4.2 폴더 (Folder)

```typescript
interface Folder {
  id: string;
  name: string;
  path: string;
  parentId: string | null;
  unreadCount: number;
  children?: Folder[];
}

// 목업 데이터
const mockFolders: Folder[] = [
  {
    id: 'folder-1',
    name: '업무',
    path: '/업무',
    parentId: null,
    unreadCount: 12,
    children: [
      {
        id: 'folder-1-1',
        name: '프로젝트A',
        path: '/업무/프로젝트A',
        parentId: 'folder-1',
        unreadCount: 5,
        children: [],
      },
      {
        id: 'folder-1-2',
        name: '프로젝트B',
        path: '/업무/프로젝트B',
        parentId: 'folder-1',
        unreadCount: 7,
        children: [],
      },
    ],
  },
  {
    id: 'folder-2',
    name: '개인',
    path: '/개인',
    parentId: null,
    unreadCount: 3,
    children: [
      {
        id: 'folder-2-1',
        name: '쇼핑',
        path: '/개인/쇼핑',
        parentId: 'folder-2',
        unreadCount: 3,
        children: [],
      },
    ],
  },
  {
    id: 'folder-3',
    name: '뉴스레터',
    path: '/뉴스레터',
    parentId: null,
    unreadCount: 8,
    children: [],
  },
  {
    id: 'folder-4',
    name: '미분류',
    path: '/미분류',
    parentId: null,
    unreadCount: 2,
    children: [],
  },
];
```

### 4.3 메일 (Mail)

```typescript
interface Mail {
  id: string;
  subject: string;
  from: {
    name: string;
    email: string;
  };
  to: {
    name: string;
    email: string;
  }[];
  body: string;
  bodyPreview: string;
  folderId: string;
  folderPath: string;
  isRead: boolean;
  isStarred: boolean;
  receivedAt: string;
  classificationReason?: string;
}

// 목업 데이터
const mockMails: Mail[] = [
  {
    id: 'mail-1',
    subject: '[프로젝트A] 주간 회의록 공유드립니다',
    from: { name: '김팀장', email: 'kim@company.com' },
    to: [{ name: '홍길동', email: 'user@example.com' }],
    body: `안녕하세요, 홍길동님

이번 주 회의록을 공유드립니다.

## 주요 논의 사항
1. 신규 기능 개발 일정 확정
2. QA 테스트 계획 수립
3. 다음 스프린트 백로그 정리

## 액션 아이템
- [ ] 기획서 검토 (홍길동)
- [ ] API 명세서 작성 (박개발)
- [ ] 디자인 시안 전달 (이디자인)

감사합니다.`,
    bodyPreview: '안녕하세요, 홍길동님. 이번 주 회의록을 공유드립니다...',
    folderId: 'folder-1-1',
    folderPath: '/업무/프로젝트A',
    isRead: false,
    isStarred: true,
    receivedAt: '2025-12-10T09:30:00Z',
    classificationReason: '프로젝트A 관련 회의록으로 분류됨',
  },
  {
    id: 'mail-2',
    subject: 'PR Review 요청: feat/email-classification',
    from: { name: 'GitHub', email: 'noreply@github.com' },
    to: [{ name: '홍길동', email: 'user@example.com' }],
    body: `@developer님이 PR 리뷰를 요청했습니다.

**Pull Request #42**
feat: LLM 기반 메일 분류 로직 구현

변경 사항:
- OpenAI API 연동
- 분류 알고리즘 구현
- 단위 테스트 추가

[PR 보러가기](https://github.com/example/pigeon/pull/42)`,
    bodyPreview: '@developer님이 PR 리뷰를 요청했습니다...',
    folderId: 'folder-1-2',
    folderPath: '/업무/프로젝트B',
    isRead: false,
    isStarred: false,
    receivedAt: '2025-12-10T08:15:00Z',
    classificationReason: 'GitHub 알림으로 업무/프로젝트B로 분류됨',
  },
  {
    id: 'mail-3',
    subject: '주문하신 상품이 배송 완료되었습니다',
    from: { name: '쿠팡', email: 'no-reply@coupang.com' },
    to: [{ name: '홍길동', email: 'user@example.com' }],
    body: `홍길동님, 안녕하세요.

주문하신 상품이 배송 완료되었습니다.

주문번호: 2024121012345
상품명: 무선 키보드
배송완료일: 2025-12-10

[배송조회](https://www.coupang.com/tracking/12345)`,
    bodyPreview: '주문하신 상품이 배송 완료되었습니다...',
    folderId: 'folder-2-1',
    folderPath: '/개인/쇼핑',
    isRead: true,
    isStarred: false,
    receivedAt: '2025-12-10T07:00:00Z',
    classificationReason: '쇼핑몰 배송 알림으로 개인/쇼핑으로 분류됨',
  },
  {
    id: 'mail-4',
    subject: 'TechNews Weekly #128 - AI 트렌드 총정리',
    from: { name: 'TechNews', email: 'newsletter@technews.com' },
    to: [{ name: '홍길동', email: 'user@example.com' }],
    body: `이번 주 테크 뉴스 하이라이트

## AI 트렌드
- Claude 4 출시 임박
- GPT-5 루머 정리
- 오픈소스 LLM 성능 비교

## 개발자 도구
- VS Code 새 기능
- GitHub Copilot 업데이트

[전체 뉴스레터 보기](https://technews.com/weekly/128)`,
    bodyPreview: '이번 주 테크 뉴스 하이라이트...',
    folderId: 'folder-3',
    folderPath: '/뉴스레터',
    isRead: true,
    isStarred: false,
    receivedAt: '2025-12-09T18:00:00Z',
    classificationReason: '정기 뉴스레터로 분류됨',
  },
  {
    id: 'mail-5',
    subject: '안녕하세요, 문의드립니다',
    from: { name: 'Unknown', email: 'unknown@example.org' },
    to: [{ name: '홍길동', email: 'user@example.com' }],
    body: `안녕하세요.

일반적인 문의 메일입니다.

감사합니다.`,
    bodyPreview: '안녕하세요. 일반적인 문의 메일입니다...',
    folderId: 'folder-4',
    folderPath: '/미분류',
    isRead: false,
    isStarred: false,
    receivedAt: '2025-12-09T15:30:00Z',
    classificationReason: '분류 기준에 맞지 않아 미분류로 이동',
  },
];
```

### 4.4 동기화 상태 (SyncStatus)

```typescript
interface SyncStatus {
  status: 'idle' | 'syncing' | 'classifying' | 'completed' | 'error';
  progress: number; // 0-100
  totalMails: number;
  processedMails: number;
  lastSyncAt: string | null;
  errorMessage?: string;
}

// 목업 데이터
const mockSyncStatus: SyncStatus = {
  status: 'completed',
  progress: 100,
  totalMails: 50,
  processedMails: 50,
  lastSyncAt: '2025-12-10T10:00:00Z',
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
  isAuthenticated: boolean;
  isLoading: boolean;
  login: () => Promise<void>;
  logout: () => Promise<void>;
}

// stores/folderStore.ts
interface FolderStore {
  folders: Folder[];
  selectedFolderId: string | null;
  isLoading: boolean;
  fetchFolders: () => Promise<void>;
  selectFolder: (id: string) => void;
}

// stores/mailStore.ts
interface MailStore {
  mails: Mail[];
  selectedMailId: string | null;
  isLoading: boolean;
  fetchMails: (folderId?: string) => Promise<void>;
  selectMail: (id: string) => void;
  markAsRead: (id: string) => Promise<void>;
  deleteMail: (id: string) => Promise<void>;
  moveMail: (id: string, folderId: string) => Promise<void>;
}

// stores/syncStore.ts
interface SyncStore {
  status: SyncStatus;
  startSync: () => Promise<void>;
  pollStatus: () => Promise<void>;
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
┌─────────────────────────────────────────────────────────────────┐
│ 🕊️ Pigeon          🔄 동기화    user@example.com ▼  [로그아웃]   │
├────────────┬─────────────────────┬──────────────────────────────┤
│ 📁 폴더    │ 📬 메일목록         │ 📄 메일 상세                  │
│            │                     │                              │
│ ▼ 업무 (12)│ ☐ 전체선택  🗑️삭제  │ From: 김팀장                 │
│   ├ 프로젝A│ ─────────────────── │ To: 홍길동                   │
│   └ 프로젝B│ ● 주간 회의록...    │ Subject: [프로젝트A] 주간... │
│ ▼ 개인 (3) │   김팀장 | 09:30    │ ─────────────────────────────│
│   └ 쇼핑   │ ─────────────────── │                              │
│ ▶ 뉴스레터 │ ○ PR Review 요청... │ 안녕하세요, 홍길동님         │
│ ▶ 미분류   │   GitHub | 08:15   │                              │
│            │ ─────────────────── │ 이번 주 회의록을 공유...     │
│            │ ○ 배송 완료...      │                              │
│            │   쿠팡 | 07:00     │ ─────────────────────────────│
│            │                     │ 📁 업무/프로젝트A            │
│            │                     │ 💡 프로젝트A 관련 회의록...  │
│            │                     │                              │
│            │                     │ [📁이동] [🗑️삭제]            │
├────────────┴─────────────────────┴──────────────────────────────┤
│ ✓ 동기화 완료 | 총 50개 메일 | 마지막 확인: 30초 전              │
└─────────────────────────────────────────────────────────────────┘
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
