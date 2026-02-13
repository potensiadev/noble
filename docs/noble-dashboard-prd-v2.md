# Noble 대시보드 — PRD v2.0 (프론트엔드 상세 설계)

> **Document Status:** Final Draft (v2.2)
> **Last Updated:** 2025-02-13
> **Depends On:** Noble PRD v2.2 (noble-prd-v2.md)
> **Phase:** Phase 0 MVP
> **주요 변경 (v2.2):** 40개 기술 스펙 확정 반영, SWR 캐시 전략, Rate Limiting 정의

---

## 1. 설계 원칙

| # | 원칙 | 이유 |
|---|------|------|
| 1 | **화면 5개 이하** | 1인 개발 5일 내 구현 가능 범위 |
| 2 | **배포는 항상 1클릭** | FAB 버튼이 모든 화면에 상주 |
| 3 | **Phase 0는 기능만** | 디자인 다듬기는 Phase 1 |
| 4 | **노션이 에디터** | 대시보드는 관리/모니터링 전용, 글 편집 없음 |
| 5 | **에러는 구체적으로** | "문제가 발생했습니다" 금지. 항상 원인 + 해결책 |

---

## 2. 프로젝트 구조 (Next.js App Router)

```
src/
├── app/
│   ├── (auth)/                    ← 인증 레이아웃 (사이드바 없음)
│   │   ├── login/page.tsx
│   │   ├── signup/page.tsx
│   │   └── layout.tsx             ← 센터 정렬, Noble 로고만
│   │
│   ├── (dashboard)/               ← 대시보드 레이아웃 (사이드바 있음)
│   │   ├── dashboard/page.tsx     ← 메인 + 빌드 모니터링
│   │   ├── posts/page.tsx
│   │   ├── settings/page.tsx      ← 2개 탭: 일반/결제
│   │   ├── onboarding/page.tsx    ← 2단계 위저드
│   │   ├── faq/page.tsx           ← FAQ + 이메일 문의 폼
│   │   └── layout.tsx             ← 사이드바 + 알림 배너 + FAB
│   │
│   ├── api/                       ← API Routes (PRD v2.0 섹션 4 참고)
│   │   ├── auth/callback/notion/route.ts
│   │   ├── site/route.ts
│   │   ├── notion/route.ts
│   │   ├── build/route.ts
│   │   ├── posts/route.ts
│   │   └── paddle/route.ts
│   │
│   └── layout.tsx                 ← 루트 레이아웃 (fonts, metadata)
│
├── components/
│   ├── layout/
│   │   ├── Sidebar.tsx
│   │   ├── AlertBanner.tsx
│   │   ├── DeployFAB.tsx
│   │   └── MobileNav.tsx
│   │
│   ├── onboarding/                  ← 2단계 위저드
│   │   ├── StepProgress.tsx         ← 2단계 프로그레스
│   │   ├── StepNotionConnect.tsx    ← Step 1: 연동 + DB + 속성 (통합)
│   │   └── StepUrlSetup.tsx         ← Step 2: URL + TOC 설정
│   │
│   ├── dashboard/
│   │   ├── BlogSummaryCard.tsx
│   │   ├── SeoStatusCard.tsx
│   │   └── PlanCard.tsx
│   │
│   ├── posts/
│   │   ├── PostTable.tsx
│   │   ├── PostRow.tsx
│   │   ├── SeoIndicator.tsx
│   │   └── FilterTabs.tsx
│   │
│   ├── settings/                    ← 2개 탭
│   │   ├── SettingsTabs.tsx         ← URL query 기반 탭 전환
│   │   ├── GeneralTab.tsx           ← 이름, 도메인, 노션, SEO, TOC
│   │   └── BillingTab.tsx           ← 체험/Pro/취소/환불
│   │
│   └── ui/                        ← 공통 UI 컴포넌트
│       ├── Button.tsx
│       ├── Input.tsx
│       ├── Toast.tsx
│       ├── Modal.tsx
│       ├── Badge.tsx
│       ├── Spinner.tsx
│       ├── CopyButton.tsx
│       └── Tooltip.tsx
│
├── hooks/
│   ├── useUser.ts                 ← Supabase Auth 상태
│   ├── useSite.ts                 ← 사이트 데이터 (SWR)
│   ├── usePosts.ts                ← 포스트 목록 (SWR)
│   ├── useBuild.ts                ← 빌드 상태 + 폴링
│   ├── useDeployLimit.ts          ← 배포 횟수 제한 상태
│   └── useToast.ts                ← 토스트 알림 관리
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts              ← 브라우저용 클라이언트
│   │   └── server.ts              ← 서버용 클라이언트
│   ├── paddle.ts                  ← Paddle.js 초기화
│   ├── encryption.ts              ← AES-256-GCM (서버 전용)
│   └── utils.ts                   ← 상대 시간, slug 검증 등
│
└── types/
    ├── database.ts                ← Supabase 생성 타입
    └── api.ts                     ← API request/response 타입
```

### 2.1 핵심 의존성

```json
{
  "dependencies": {
    "next": "^14.2",
    "@supabase/supabase-js": "^2.39",
    "@supabase/ssr": "^0.1",
    "swr": "^2.2",
    "lucide-react": "^0.312",
    "@paddle/paddle-js": "^1.0",
    "clsx": "^2.1",
    "tailwind-merge": "^2.2"
  },
  "devDependencies": {
    "tailwindcss": "^3.4",
    "typescript": "^5.3",
    "@types/react": "^18"
  }
}
```

| 라이브러리 | 용도 | 선택 이유 |
|-----------|------|----------|
| **SWR** | 서버 데이터 캐시/페칭 | 자동 재검증, 경량 (React Query 대비 번들 작음) |
| **Lucide React** | 아이콘 | 트리 셰이킹 지원, shadcn/ui 호환 |
| **Tailwind CSS** | 스타일링 | 유틸리티 우선, 빠른 프로토타이핑 |
| **clsx + tailwind-merge** | 조건부 클래스 | Tailwind 클래스 충돌 해결 |

---

## 3. 라우팅 & 인증 가드

### 3.1 라우팅 테이블

| 경로 | 페이지 | 인증 필요 | 온보딩 필요 | 레이아웃 |
|------|-------|----------|-----------|---------|
| `/login` | 로그인 | ❌ | ❌ | AuthLayout (센터) |
| `/signup` | 회원가입 | ❌ | ❌ | AuthLayout (센터) |
| `/onboarding` | 온보딩 위저드 | ✅ | ❌ (진행 중) | DashboardLayout (사이드바 숨김) |
| `/dashboard` | 메인 대시보드 | ✅ | ✅ | DashboardLayout |
| `/posts` | 포스트 관리 | ✅ | ✅ | DashboardLayout |
| `/settings` | 설정 | ✅ | ✅ | DashboardLayout |
| `/settings?tab=billing` | 결제 설정 | ✅ | ✅ | DashboardLayout |

### 3.2 미들웨어 라우팅 로직

```typescript
// middleware.ts
export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const session = await getSession(request);

  // 1. 비인증 사용자 → 보호된 경로 접근 시 로그인으로
  if (!session && isProtectedRoute(pathname)) {
    return redirect('/login');
  }

  // 2. 인증된 사용자 → 로그인/회원가입 접근 시 대시보드로
  if (session && isAuthRoute(pathname)) {
    return redirect('/dashboard');
  }

  // 3. 온보딩 미완료 → 대시보드/포스트/설정 접근 시 온보딩으로
  if (session && !session.user.onboarding_completed && isDashboardRoute(pathname)) {
    return redirect('/onboarding');
  }

  // 4. 온보딩 완료 → /onboarding 접근 시 대시보드로
  if (session && session.user.onboarding_completed && pathname === '/onboarding') {
    return redirect('/dashboard');
  }

  return NextResponse.next();
}
```

### 3.3 체험 만료 접근 제어

```
체험 만료 시:
  - /dashboard: 접근 허용 (업그레이드 유도 카드 표시)
  - /posts: 접근 허용 (읽기 전용, 배포 버튼 비활성)
  - /settings: 접근 허용 (결제 탭으로 유도)
  - 배포 API: 402 TRIAL_EXPIRED 반환
  - 블로그: 서빙 중지 (Cloudflare에서 "구독이 필요합니다" 페이지로 대체)
```

---

## 4. 화면 1: 로그인/회원가입 (`/login`, `/signup`)

### 4.1 레이아웃

```
┌──────────────────────────────────────────────────┐
│                                                  │
│                   Noble 로고                      │
│                                                  │
│            "노션으로 글만 쓰면,                     │
│             구글에 잘 잡히는 블로그"                  │
│                                                  │
│         ┌──────────────────────────┐             │
│         │  📧 이메일로 시작하기      │             │
│         └──────────────────────────┘             │
│                                                  │
│         [이메일 입력 폼 - 조건부 표시]               │
│         ┌──────────────────────────┐             │
│         │  email@example.com       │             │
│         └──────────────────────────┘             │
│         [ 매직 링크 전송 ]                         │
│                                                  │
│         ── 또는 ──                                │
│                                                  │
│         ┌──────────────────────────┐             │
│         │  🔗 Google로 시작하기      │             │
│         └──────────────────────────┘             │
│                                                  │
│          14일 무료 체험 · 카드 등록 불필요           │
│                                                  │
│        이미 계정이 있으신가요? 로그인                │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 4.2 컴포넌트 상세

| 요소 | 컴포넌트 | 스펙 |
|------|---------|------|
| **로고** | `<img>` | Noble 로고, 높이 40px, 중앙 정렬 |
| **태그라인** | `<h1>` | "노션으로 글만 쓰면, 구글에 잘 잡히는 블로그" |
| **이메일 버튼** | `<Button>` | 클릭 → 이메일 입력 폼 토글 표시 |
| **이메일 입력** | `<Input type="email">` | placeholder: "email@example.com" |
| **매직 링크 전송** | `<Button variant="primary">` | 클릭 → Supabase `signInWithOtp()` |
| **Google 버튼** | `<Button variant="outline">` | 클릭 → Supabase `signInWithOAuth({ provider: 'google' })` |
| **신뢰 문구** | `<p>` | "14일 무료 체험 · 카드 등록 불필요" |
| **전환 링크** | `<Link>` | 회원가입 ↔ 로그인 토글 |
| **AuthLayout** | 래퍼 | 화면 중앙 정렬, 최대 너비 400px, 사이드바/FAB 없음 |

### 4.3 상태 머신

```
[idle]
  ├── 이메일 버튼 클릭 → [email_form_visible]
  │     ├── 이메일 입력 + 전송 클릭 → [sending_magic_link]
  │     │     ├── 성공 → [magic_link_sent] "이메일을 확인해주세요" 표시
  │     │     └── 실패 → [email_error] 에러 메시지 표시
  │     └── 이메일 형식 오류 → [email_validation_error]
  │
  └── Google 버튼 클릭 → [google_redirect] OAuth 리다이렉트
```

### 4.4 에러 처리

| 에러 | 발생 조건 | 사용자 메시지 |
|------|----------|-------------|
| 이메일 형식 오류 | `@` 없음, 도메인 없음 | "올바른 이메일 주소를 입력해주세요" |
| Magic Link 전송 실패 | Supabase API 에러 | "이메일 전송에 실패했습니다. 잠시 후 다시 시도해주세요" |
| Magic Link rate limit | 60초 내 재전송 | "잠시 후 다시 시도해주세요 (N초)" |
| Google OAuth 실패 | 팝업 차단, 네트워크 | "Google 로그인에 실패했습니다. 다시 시도해주세요" |

### 4.5 Magic Link 전송 후 화면

```
┌──────────────────────────────────────────────────┐
│                                                  │
│                   ✉️                              │
│                                                  │
│          이메일을 확인해주세요                      │
│                                                  │
│    user@example.com으로                           │
│    로그인 링크를 보내드렸습니다.                     │
│                                                  │
│    이메일이 오지 않나요?                            │
│    [다시 보내기] (60초 쿨다운)                      │
│                                                  │
│    [← 다른 방법으로 로그인]                         │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 4.6 회원가입 시 서버 로직

```
1. Supabase Auth 계정 생성 (이메일 또는 Google)
2. auth.users에 자동 생성
3. DB trigger 또는 API에서:
   INSERT INTO public.users (id, email, plan, trial_ends_at)
   VALUES (auth.uid(), email, 'trial', NOW() + INTERVAL '14 days')
4. onboarding_completed = FALSE → 로그인 시 /onboarding으로 리다이렉트
```

---

## 5. 화면 2: 온보딩 위저드 (`/onboarding`)

### 5.1 전체 구조

```
레이아웃: DashboardLayout 사용하되 사이드바 숨김 (온보딩 전용 모드)
FAB: 숨김 (아직 사이트 없음)
알림 배너: 숨김
진행 상태: URL query parameter (?step=1,2,3,4)
데이터 저장: 각 단계 완료 시 서버에 즉시 저장 (중간 이탈 시 복원 가능)
```

### 5.2 온보딩 상태 관리 (2단계)

```typescript
interface OnboardingState {
  currentStep: 1 | 2;
  step1: {
    // Notion 연동 + DB 선택 + 속성 검증 (통합)
    completed: boolean;
    notionConnected: boolean;
    accessToken: string | null;     // 메모리 only, DB에는 암호화 저장
    workspaceId: string | null;
    workspaceName: string | null;
    selectedDatabaseId: string | null;
    selectedDatabaseName: string | null;
    databases: Array<{ id: string; title: string; pageCount: number }>;
    properties: {
      title: { exists: boolean };
      slug: { exists: boolean; type: string | null };
      status: { exists: boolean; hasPublishedOption: boolean };
    };
  };
  step2: {
    // URL 설정 + TOC 설정
    completed: boolean;
    subdomain: string | null;
    customDomain: string | null;
    subdomainAvailable: boolean | null;
    dnsVerified: boolean | null;
    tocEnabled: boolean;            // 목차 자동 생성 여부 (기본: true)
  };
}

// 서버 저장: users 테이블에 onboarding_step INTEGER 컬럼 추가
// 세부 데이터는 sites 테이블에 즉시 저장
```

### 5.3 프로그레스 바 (`StepProgress`) — 2단계

```
표시:  ● ─────────── ○
      연동+DB         URL

상태별 스타일:
  ● 완료됨: 채워진 원, primary 색상
  ● 현재:   채워진 원, primary 색상, 펄스 애니메이션
  ○ 미완료: 빈 원, gray-300
  ─── 연결선: 완료 구간 primary, 미완료 gray-200
모바일: 라벨 text-xs

기존 4단계(연동/DB/속성/URL) → 2단계로 간소화
```

### 5.4 Step 1: Notion 연동 + DB 선택 + 속성 검증 (`StepNotionConnect`)

> 기존 Step 1~3을 하나로 통합. 한 화면에서 연동 → DB 선택 → 속성 검증 순차 진행.

#### 레이아웃

```
┌────────────────────────────────────────────────┐
│  📌 노션과 연결하기                              │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ 1️⃣ 노션 연동   [🔗 노션 연동하기]         │  │ ← 완료 시 ✅
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ 2️⃣ DB 선택                               │  │ ← 연동 후 표시
│  │   ○ 블로그 포스트 (12개)                   │  │
│  │   ○ 개발 노트 (5개)                        │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │ 3️⃣ 속성 확인                              │  │ ← DB 선택 후 자동 검증
│  │   ✅ Title  ✅ Slug  ✅ Status             │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ▷ 연동이 안 되시나요? (URL 폴백)               │
│                                                │
│  [다음] (모든 ✅ 시 활성화)                     │
└────────────────────────────────────────────────┘
```

#### OAuth 플로우

```
1. [노션 연동하기] 클릭 → window.location.href = Notion OAuth URL
   https://api.notion.com/v1/oauth/authorize
     ?client_id={NOTION_CLIENT_ID}
     &response_type=code
     &owner=user
     &redirect_uri={BASE_URL}/api/auth/callback/notion
     &state={csrf_token_from_cookie}

2. 사용자: Notion 화면에서 DB 선택 + 승인

3. Notion → /api/auth/callback/notion?code={code}&state={state}

4. 서버:
   a. state 쿠키와 query state 비교 (CSRF 검증)
   b. POST https://api.notion.com/v1/oauth/token
   c. access_token AES-256-GCM 암호화 → 임시 저장
   d. redirect → /onboarding?step=1&connected=true (DB 선택 섹션 활성화)
```

#### DB 선택 + 속성 검증 (인라인)

| 상태 | UI |
|------|-----|
| DB 0개 | "연동된 데이터베이스가 없습니다" + [템플릿 복제하기] |
| DB 1개 | 자동 선택 → 속성 자동 검증 |
| DB 2개+ | 라디오 목록, 선택 시 속성 자동 검증 |
| 속성 누락 | 구체적 안내 + [다시 확인하기] |
| 모든 ✅ | [다음] 버튼 활성화 |

### 5.5 Step 2: 블로그 URL + TOC 설정 (`StepUrlSetup`)

#### 레이아웃

```
┌────────────────────────────────────────────────┐
│  🌐 블로그 URL 설정                             │
│                                                │
│  서브도메인                                     │
│  ┌────────────────────────┐                    │
│  │ myblog                 │.noble.blog        │
│  └────────────────────────┘                    │
│  ✅ 사용 가능                                   │
│                                                │
│  ▷ 커스텀 도메인 (선택사항)                      │
│  ┌────────────────────────────────────────┐    │
│  │ blog.mydomain.com                      │    │
│  └────────────────────────────────────────┘    │
│  CNAME: noble-cdn.pages.dev                    │
│  [DNS 확인하기]                                 │
│                                                │
│  ─────────────────────────────────────         │
│                                                │
│  📋 블로그 설정                                 │
│  ☑️ 목차 자동 생성 (권장)                        │ ← TOC 설정
│     h1/h2/h3 기반 목차를 자동으로 만듭니다       │
│                                                │
│  [🚀 블로그 만들기]                              │
└────────────────────────────────────────────────┘
```

서브도메인:
```
규칙: 3~30자, 영문 소문자+숫자+하이픈, 시작/끝 하이픈 불가
예약어: admin, api, www, app, dashboard, blog, help, support, status, mail, noble
검증: 클라이언트 regex + 서버 DB unique (debounce 500ms)
```

커스텀 도메인:
```
접힘 토글, 선택사항
DNS 안내: CNAME → noble-cdn.pages.dev
[DNS 확인하기] → POST /api/site/verify-domain → DNS lookup
⏳ "DNS 전파에 최대 48시간 걸릴 수 있습니다"
```

TOC 설정:
```
☑️ 목차 자동 생성 (기본값: 켜짐)
→ sites.toc_enabled = true/false
→ 설정 > 일반에서 언제든 변경 가능
```

**"블로그 만들기" 클릭 시 (멱등성 보장):**
```
1. 클릭 즉시: 버튼 → "생성 중..." (disabled, 더블클릭 방지)
2. POST /api/site (사이트 생성 + toc_enabled + Cloudflare 프로젝트 + 첫 빌드 트리거)
   - 서버: user_id로 기존 사이트 존재 확인
   - 이미 존재 → 기존 사이트 반환 (중복 생성 안 함)
   - DB UNIQUE 제약 (one_site_per_user)이 최종 방어
3. users.onboarding_completed = TRUE
4. /dashboard 리다이렉트
5. 대시보드에서 빌드 폴링 시작 (Workers에서 실행)
6. 완료 시: "🎉 축하합니다! 블로그가 생성되었습니다!" 토스트
```

### 5.6 온보딩 복원

```
브라우저 이탈 후 재방문:
  users.onboarding_step 확인 → 해당 단계부터 재시작
  이전 단계 데이터 서버에 저장되어 있으므로 ✅ 유지
```

---

## 6. 대시보드 레이아웃 (`DashboardLayout`)

### 6.1 전체 구조

```
┌─────────────────────────────────────────────────────┐
│ [AlertBanner — 조건부]                               │
├──────────┬──────────────────────────────────────────┤
│ Sidebar  │            Main Content                  │
│ (240px)  │            (flex-1, p-6 lg:p-8)          │
│          │                                          │
│          │                          ┌──────┐       │
│          │                          │ FAB  │       │
│          │                          └──────┘       │
└──────────┴──────────────────────────────────────────┘
```

### 6.2 사이드바 (`Sidebar`)

반응형:
| 브레이크포인트 | 너비 | 동작 |
|-------------|------|------|
| 1024px+ | 240px 고정 | 아이콘 + 텍스트, 항상 표시 |
| 768~1023px | 64px 고정 | 아이콘만, 호버 시 툴팁 |
| ~767px | 0px (숨김) | 상단 햄버거 → 오버레이 슬라이드 (280px, bg-black/50) |

메뉴 구조:
```
[Noble 로고] → /dashboard
───
📊 대시보드     /dashboard
📝 포스트       /posts
⚙️ 설정        /settings
───
🔗 내 블로그    외부 링크 (새탭) — 빌드 전이면 disabled
📖 도움말       외부 링크 (새탭)
───
Pro 플랜 (또는 ⏱️ 체험 D-N)
───
user@email
[로그아웃]
```

활성 메뉴: `pathname === href` (대시보드) 또는 `pathname.startsWith(href)` (설정)

### 6.3 알림 배너 (`AlertBanner`)

우선순위 순 (1개만 표시):

| P | 조건 | 타입 | 메시지 | CTA | 추가 액션 |
|---|------|------|-------|-----|----------|
| 1 | `last_build_status === 'failed'` | error | "마지막 배포가 실패했습니다" | [상세 보기] | - |
| 2 | Notion API 401 감지 | warning | "노션 연동이 해제되었습니다" | [재연동하기] | **자동 이메일** |
| 3 | `paddle_subscription_status === 'past_due'` | warning | "결제에 실패했습니다" | [결제 수단 확인] | - |
| 4 | trial D-3 | warning | "체험 기간이 3일 남았습니다" | [업그레이드] | **푸시 이메일** |
| 5 | trial D-1 | error | "체험 기간이 내일 만료됩니다" | [업그레이드] | **푸시 이메일** |
| 6 | trial 만료 | error | "체험 기간이 만료되었습니다" | [업그레이드] | - |

> 7일 체험 + D-3, D-1 푸시 강화

스타일:
```
error:   bg-red-50    border-l-4 border-red-500    text-red-800
warning: bg-yellow-50 border-l-4 border-yellow-500 text-yellow-800
info:    bg-blue-50   border-l-4 border-blue-500   text-blue-800
닫기 버튼 없음 (상태 해결 시 자동 소멸)
```

### 6.4 배포 FAB (`DeployFAB`) — 3개 상태

위치: fixed, bottom-6 right-6, z-40
데스크톱: 48px 높이 + "🚀 배포" 텍스트, rounded-full
모바일: 56×56px 원형, 아이콘만

#### 3개 상태 (기존 5개에서 간소화)

| 상태 | 아이콘 | 텍스트 | 배경 | 클릭 |
|------|-------|-------|------|------|
| `idle` | Rocket | "배포" | blue-600 | 빌드 트리거 |
| `building` | Spinner | "배포 중..." | gray-400 | disabled |
| `done` | Check/X | "완료!"/"실패" | green/red-600 | 3초→idle (실패 시 에러 모달) |

> limited/expired 상태는 클릭 시 토스트로 안내 ("오늘 배포 횟수 초과", "체험 기간 만료")

호버 툴팁: "오늘 남은 배포: {limit-used}/{limit}"

#### 클릭 → 빌드 플로우 (멱등성 보장)

```
1. 클릭 즉시: FAB → building 상태 (disabled, 더블클릭 방지)
2. Idempotency-Key 생성: UUID v4 → 헤더에 포함
3. POST /api/build/trigger (Idempotency-Key: {uuid})
   → Vercel API가 Cloudflare Workers 호출 (빌드 파이프라인 분리)
4. 응답 분기:
   - 200 → build_id → 폴링 시작
   - 429 → "오늘 배포 횟수 초과" 토스트 (FAB는 idle 유지)
   - 402 → "체험 기간 만료" 토스트 (FAB는 idle 유지)
   - 409 → "이미 빌드 진행 중" 토스트 + existing_build_id로 폴링 시작
   ※ 동일 Idempotency-Key로 재요청 시 → 이전 응답 반환 (새 빌드 생성 안 함)
```

#### 폴링 전략

```typescript
0~30초:   2초 간격
30~120초: 5초 간격
120~300초: 10초 간격
300초+:   폴링 중단, "빌드에 시간이 걸리고 있습니다" 표시
```

---

## 7. 화면 3: 메인 대시보드 (`/dashboard`)

### 7.1 데이터

```
SWR 캐시 전략 (엔드포인트별):
- GET /api/site: refreshInterval 30초, revalidateOnFocus: true
- GET /api/posts: refreshInterval 60초, revalidateOnFocus: true
- GET /api/build/status: 폴링 (0-30초: 2초, 30-120초: 5초, 120-300초: 10초)
- GET /api/build/history: refreshInterval 없음 (수동 재검증)
```

### 7.2 레이아웃 (5개 카드)

```
[BlogSummaryCard]     ← 발행됨 / 초안 / 마지막 배포 / 상태
[BuildMonitorCard]    ← 빌드 성공률 / 평균 빌드 시간 / 최근 빌드 (신규)
[SeoStatusCard]       ← sitemap, RSS 복사 + 서치콘솔 가이드
[PlanCard]            ← 체험 남은 일 (7일) / Pro 플랜 / 만료 상태
[통계 placeholder]    ← "페이지뷰 통계가 곧 추가됩니다" (Phase 1)
```

### 7.3 BlogSummaryCard

4개 지표 가로 배열 (모바일: 2×2 그리드):

| 지표 | 소스 | 표시 |
|------|------|------|
| 발행된 글 | posts count (Published) | "12개" |
| 초안 | posts count (Draft) | "3개" |
| 마지막 배포 | site.last_build_at | 상대 시간 |
| 블로그 상태 | site.last_build_status | 🟢정상 / 🟡빌드중 / 🔴실패 / ⚪빌드전 |

빌드 전 상태 (온보딩 직후):
```
🟡 첫 번째 블로그를 만들고 있습니다...
████████████████░░░░░░ 65%
노션에서 글을 가져오는 중... (15/32)
```

### 7.4 BuildMonitorCard (신규)

빌드 모니터링 대시보드 — 빌드 실패율, 평균 빌드 시간 표시.

| 지표 | 데이터 소스 | 표시 형식 |
|------|-----------|----------|
| 빌드 성공률 | builds 최근 30일 | "95%" + 원형 프로그레스 |
| 평균 빌드 시간 | builds.build_duration_ms 평균 | "23초" |
| 최근 빌드 | builds 최근 5개 | 미니 타임라인 (✅❌⏳) |
| 서비스 상태 | UptimeRobot API | 🟢 정상 / 🟡 점검 중 |

```
┌─────────────────────────────────────────────────┐
│  📊 빌드 모니터링                                │
├─────────────────────────────────────────────────┤
│                                                 │
│   [🟢 95%]        평균 빌드: 23초               │
│   빌드 성공률                                    │
│                                                 │
│   최근 빌드: ✅ ✅ ✅ ❌ ✅                        │
│                                                 │
│   서비스 상태: 🟢 정상 (UptimeRobot)             │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 7.5 SeoStatusCard

| 요소 | 동작 |
|------|------|
| Sitemap URL | `https://{domain}/sitemap.xml` + [📋 복사] → 클립보드 + 토스트 |
| RSS URL | `https://{domain}/rss.xml` + [📋 복사] → 동일 |
| 서치콘솔 가이드 | Google + Naver 외부 링크 |
| 빌드 전 | "첫 배포 후 사용할 수 있습니다" |

### 7.6 PlanCard (Phase 0: Pro만)

| 상태 | 표시 |
|------|------|
| **체험 중** | "⏱️ 7일 무료 체험 중" + 남은 N일 + 프로그레스 바 + [Pro 시작하기] |
| **체험 D-3** | 주황 배경, "체험 기간이 3일 남았습니다" + [지금 업그레이드] |
| **체험 D-1** | 빨간 배경, "내일 체험이 만료됩니다" + [지금 업그레이드] |
| **체험 만료** | 빨간 배경, Pro 플랜 카드 + [시작하기] |
| **Pro 정상** | "🟢 Pro 플랜 · $9/월 · 다음 결제일: YYYY.MM.DD" |
| **결제 실패** | 주황 배경, "마지막 결제가 실패했습니다" + [결제 수단 업데이트] |

> Business 플랜은 Phase 1에서 추가

### 7.6 빌드 에러 모달

FAB `failed` 클릭 또는 알림 배너 "상세 보기" 클릭 시:

```
┌──────────────────────────────────────┐
│  ❌ 배포 실패                        │
│  에러: {error_message}               │
│  발생 단계: {error_step}             │
│  시간: {completed_at}                │
│                                      │
│  해결 방법:                           │
│  {에러별 안내 — 아래 참고}            │
│                                      │
│  [다시 배포하기]    [닫기]             │
└──────────────────────────────────────┘
```

| error_step | 안내 |
|-----------|------|
| `notion_auth` | "노션 재연동이 필요합니다" + [재연동] |
| `notion_fetch` | "노션에서 데이터를 가져오지 못했습니다. 잠시 후 다시 시도해주세요" |
| `image_processing` | "일부 이미지 처리에 실패했습니다. 다시 배포하면 해결될 수 있습니다" |
| `html_generation` | "HTML 생성 중 오류. 노션 글 내용을 확인해주세요" |
| `cloudflare_deploy` | "Cloudflare 배포 실패. 잠시 후 다시 시도해주세요" |
| `timeout` | "빌드 시간 초과. 글이 너무 많으면 일부를 Draft로 변경해주세요" |

---

## 8. 화면 4: 포스트 관리 (`/posts`)

### 8.1 데이터

```
API: GET /api/posts?status=all|published|draft
데이터: 마지막 빌드 시 캐시된 posts 테이블 (실시간 아님)
Response: { posts: [...], counts: { all, published, draft } }
```

### 8.2 레이아웃

```
📝 포스트 관리

[전체 15] [발행됨 12] [초안 3]              ← FilterTabs

┌──────────────────────────────────────────┐
│ 📄 Next.js 시작하기                       │ ← 클릭 → Notion 새탭
│    nextjs-getting-started                │
│    Published · 2시간 전 · 5분 읽기       │
│    SEO: ✅ meta  ✅ slug  ✅ image        │
├──────────────────────────────────────────┤
│ 📄 React 상태관리                         │
│    react-state-management                │
│    Published · 1일 전 · 12분 읽기        │
│    SEO: ✅ meta  ✅ slug  ⚠️ image없음   │
├──────────────────────────────────────────┤
│ 📄 GraphQL 입문                           │
│    (slug 없음)                           │ ← 주황 이탤릭
│    Draft · 3일 전                        │
│    SEO: ⚠️ meta  ❌ slug없음  ⚠️ image  │
└──────────────────────────────────────────┘

💡 글 편집은 노션에서 직접 하세요.
   편집 후 배포 버튼을 누르면 반영됩니다.
```

### 8.3 FilterTabs

```
활성: bg-primary text-white font-semibold rounded-full px-4 py-1
비활성: bg-gray-100 text-gray-600 rounded-full px-4 py-1
```

### 8.4 PostRow

| 요소 | 스타일 | 동작 |
|------|-------|------|
| 제목 | text-base font-medium text-gray-900 | 전체 행 클릭 → Notion 새탭 |
| Slug | text-sm text-gray-400 | 없으면 "(slug 없음)" 주황 이탤릭 |
| Status | Badge | Published: green / Draft: gray |
| 수정일 | text-sm text-gray-400 | 상대 시간 |
| 읽기 시간 | text-sm text-gray-400 | "N분 읽기" (Published만) |
| SEO | SeoIndicator | ✅/⚠️/❌ |
| 호버 | bg-gray-50 cursor-pointer | ↗ 아이콘 |

### 8.5 SeoIndicator

```
✅ CheckCircle (green-500): meta_description 10자+, slug 존재, 커버이미지 존재
⚠️ AlertTriangle (yellow-500): 권장 미충족 (이미지 없음 등)
❌ XCircle (red-500): 필수 미충족 (slug 없음)
```

### 8.6 빈 상태 & 페이지네이션

```
포스트 0개:
  📝 아이콘 + "아직 포스트가 없습니다"
  + "노션에서 글을 작성한 후 Status를 Published로 변경하고 배포하세요"
  + [노션에서 글 쓰러 가기 →]

50개 초과: [더 보기] 버튼 (무한 스크롤 아님)
정렬: 수정일 최신순 고정
```

---

## 9. 화면 5: 설정 (`/settings`) — 2개 탭

### 9.1 탭 구조

```
URL: /settings?tab=general|billing (기본: general)

탭: [일반] [결제]
```

> 기존 4개 탭(일반/SEO/테마/결제)에서 2개로 간소화. SEO는 일반에 통합.

### 9.2 일반 탭 (`GeneralTab`)

| 필드 | 타입 | 규칙 | DB 컬럼 |
|------|------|------|---------|
| 블로그 이름 | Input | 1~50자, 필수 | sites.name |
| 블로그 설명 | Textarea (3행) | 0~300자 | sites.description |
| 서브도메인 | 읽기 전용 + [변경] | 변경 시 모달 (중복 체크) | sites.subdomain |
| 커스텀 도메인 | Input | 소문자 자동, DNS 검증 | sites.custom_domain |
| 노션 연동 | 읽기 전용 | 워크스페이스+DB 이름 | - |
| **목차 설정** | 체크박스 | "목차 자동 생성" | sites.toc_enabled |

커스텀 도메인 DNS 안내:
```
Type: CNAME | Name: blog | Target: noble-cdn.pages.dev
[DNS 확인하기]
상태: ✅ 확인됨·SSL 활성 | ⏳ SSL 발급 중 | ❌ CNAME 미발견
```

노션 연동:
```
✅ 연동됨 · "My Workspace" · "블로그 포스트"
[연동 해제] → 확인 모달: "배포를 할 수 없게 됩니다"
[다른 DB 선택] → 확인 모달 → DB 선택 → 속성 검증 → 자동 빌드
```

SEO (간소화, 일반 탭에 통합):
```
Sitemap URL: https://myblog.noble.blog/sitemap.xml [📋 복사]
RSS URL: https://myblog.noble.blog/rss.xml [📋 복사]
서치콘솔 등록: [Google] [Naver] 외부 링크
```

저장: PATCH /api/site → 성공 토스트 / 에러 인라인

### 9.3 결제 탭 (`BillingTab`) — Phase 0: Pro만

#### 체험 중

```
⏱️ 7일 무료 체험 중
남은 기간: N일 | 프로그레스 바

┌─────────────────────────────────────┐
│ Pro 플랜    $9/월                    │
│ ✓ 커스텀 도메인                      │
│ ✓ 무제한 포스트                      │
│ ✓ SEO 전자동                        │
│ ✓ 일 10회 배포                      │
│ ✓ 목차 자동 생성                     │
│                                     │
│ [Pro 시작하기]                       │
│ 연간 결제 시 $86 (20% 할인)          │
└─────────────────────────────────────┘

💡 Business 플랜은 곧 출시됩니다 (애드센스, 멤버십)
```

#### Pro

```
🟢 Pro 플랜 · 월간 | $9/월 + 현지 세금 | 다음 결제일: YYYY.MM.DD
[연간 전환 20% 할인]
──
결제 관리: [🔗 Paddle 결제 관리 열기] (외부 링크)
──
[구독 취소하기] (빨간 텍스트)
──
[환불 요청하기]
```

#### Paddle Checkout 연동

```typescript
// 결제 버튼 클릭 시
Paddle.Checkout.open({
  settings: { displayMode: 'overlay', theme: 'light', locale: 'ko' },
  items: [{ priceId, quantity: 1 }],
  customer: { email: user.email },
  customData: { userId: user.id },
});

// 완료 이벤트
Paddle.Setup({
  token: PADDLE_CLIENT_TOKEN,
  eventCallback: (event) => {
    if (event.name === 'checkout.completed') {
      showToast('결제 완료! 잠시만 기다려주세요...');
      setTimeout(() => window.location.reload(), 3000);
    }
  },
});
```

#### 구독 취소 모달 (멱등성 보장)

```
"구독을 취소하시겠습니까?"
• 현재 결제 기간(YYYY.MM.DD)까지 서비스 이용 가능
• 이후 블로그 비활성
• 데이터 30일 보관
[취소하기 (빨간)]  [돌아가기 (회색)]

멱등성:
  1. [취소하기] 클릭 즉시 버튼 disabled + 스피너
  2. POST /api/paddle/cancel
  3. 서버: paddle_subscription_status 사전 체크
     → 이미 'canceled' → "이미 취소되었습니다" 반환 (Paddle 재호출 안 함)
     → 'active' → Paddle cancel API 호출 → 토스트
```

#### 환불 요청 모달 (멱등성 보장)

```
14일 이내: "전액 환불, 구독 즉시 해지"
14일 초과: "검토 후 이메일 안내"

멱등성:
  1. [환불 요청] 클릭 즉시 버튼 disabled + 스피너
  2. Idempotency-Key (UUID v4) 생성 → 헤더 포함
  3. POST /api/paddle/refund (Idempotency-Key: {uuid})
  4. 서버: 동일 키 존재 시 캐시된 응답 반환 (이중 환불 방지)
```

---

## 9.5 FAQ + 이메일 문의 (`/faq`)

1인 운영을 위한 최소 지원 채널.

```
┌─────────────────────────────────────────────────┐
│  ❓ 자주 묻는 질문                               │
├─────────────────────────────────────────────────┤
│                                                 │
│  ▶ 노션 연동이 안 돼요                           │
│    [펼치면 상세 답변]                            │
│                                                 │
│  ▶ 배포가 실패했어요                             │
│    [펼치면 상세 답변]                            │
│                                                 │
│  ▶ 커스텀 도메인 설정 방법                        │
│    [펼치면 상세 답변]                            │
│                                                 │
│  ▶ 결제/환불 문의                                │
│    [펼치면 상세 답변]                            │
│                                                 │
│  ▶ 체험 기간 연장 가능한가요?                     │
│    [펼치면 상세 답변]                            │
│                                                 │
├─────────────────────────────────────────────────┤
│  📧 문의하기                                     │
│                                                 │
│  이메일: [user@email.com    ] (자동 입력)        │
│  제목:   [                  ]                   │
│  내용:   [                                   ]  │
│          [                                   ]  │
│                                                 │
│  [문의 보내기]                                   │
│                                                 │
│  💡 1-2 영업일 내 이메일로 답변드립니다           │
└─────────────────────────────────────────────────┘
```

FAQ 항목 (Phase 0):
1. 노션 연동이 안 돼요 → URL 폴백 안내
2. 배포가 실패했어요 → 에러 유형별 해결법
3. 커스텀 도메인 설정 방법 → DNS 가이드
4. 결제/환불 문의 → Paddle 정책 안내
5. 체험 기간 연장 → "현재 지원하지 않습니다"

이메일 문의:
- POST /api/support/contact
- Resend로 관리자에게 전달
- 자동 확인 이메일 발송

---

## 10. 토스트 알림 시스템

### 10.1 스펙

```
위치: 우상단 (top-6 right-6)
최대 동시 3개 (초과 시 가장 오래된 것 제거)
z-index: 50 (FAB 위)
애니메이션: 오른쪽 슬라이드 인 (200ms)
크기: 360px (데스크톱), 풀-32px (모바일)
닫기: X 버튼 항상 표시
```

### 10.2 이벤트 목록

| 이벤트 | 타입 | 메시지 | 지속 |
|--------|------|-------|------|
| 배포 시작 | info | "배포를 시작합니다..." | 3초 |
| 배포 완료 | success | "배포 완료! 블로그 업데이트됨" + [블로그 보기] | 5초 |
| 배포 실패 | error | "배포 실패. {원인}" + [상세 보기] | 수동 닫기 |
| 횟수 초과 | warning | "오늘 배포 횟수를 모두 사용했습니다" | 5초 |
| 체험 만료 | warning | "체험 기간이 만료되었습니다" + [업그레이드] | 수동 닫기 |
| 설정 저장 | success | "설정이 저장되었습니다" | 3초 |
| 클립보드 | info | "복사되었습니다" | 2초 |
| 노션 연동 | success | "노션이 연결되었습니다!" | 3초 |
| 결제 완료 | success | "결제가 완료되었습니다!" | 5초 |
| 블로그 생성 | success | "🎉 블로그가 생성되었습니다!" | 5초 |

스타일:
```
success: bg-green-50 border-l-4 border-green-500 text-green-800
error:   bg-red-50   border-l-4 border-red-500   text-red-800
warning: bg-yellow-50 border-l-4 border-yellow-500 text-yellow-800
info:    bg-blue-50  border-l-4 border-blue-500  text-blue-800
```

---

## 11. 로딩 & 에러 상태

### 11.1 로딩

| 레벨 | UI |
|------|-----|
| 페이지 | 전체 스켈레톤 (pulse 1.5s 주기) |
| BlogSummaryCard | 4개 지표 자리 회색 블록 |
| PostTable | 3줄 행 스켈레톤 |
| 기타 카드 | 카드 높이 회색 블록 |
| 온보딩 DB 목록 | 3줄 리스트 스켈레톤 |

### 11.2 에러

네트워크 에러:
```
⚠️ 데이터를 불러오지 못했습니다
네트워크 연결을 확인해주세요
[다시 시도]
```

인증 만료 (API 401):
```
모달: "세션이 만료되었습니다. 다시 로그인해주세요"
[로그인] → /login 리다이렉트
```

전역 에러 핸들러:
```typescript
// 401 → 로그인 리다이렉트
// 402 → "업그레이드가 필요합니다" 토스트
// 429 → "잠시 후 다시 시도해주세요" 토스트
// 기타 → "문제가 발생했습니다" 토스트
```

---

## 12. 반응형 디자인

| 이름 | 너비 | 사이드바 | FAB | 특이사항 |
|------|------|---------|-----|---------|
| 모바일 | ~767px | 숨김 (햄버거) | 원형 56px | 카드 세로 스택, 포스트 제목만 |
| 태블릿 | 768~1023px | 64px (아이콘) | 원형 56px | - |
| 데스크톱 | 1024px+ | 240px (풀) | 텍스트 48px | - |

터치 최적화: 모든 인터랙티브 요소 최소 44×44px

---

## 13. 접근성 (A11y)

| 항목 | 구현 |
|------|------|
| 키보드 | 모든 요소 Tab 접근, Enter/Space 활성화 |
| 포커스 | focus-visible:ring-2 ring-blue-500 ring-offset-2 |
| ARIA | 아이콘 버튼 aria-label, 모달 role="dialog" |
| 색상 대비 | WCAG AA (4.5:1 텍스트, 3:1 UI) |
| 스크린 리더 | 상태 변경 aria-live="polite" |
| 모달 | 포커스 트랩, ESC 닫기 |

---

## 14. 성능 목표

| 지표 | 목표 |
|------|------|
| FCP | < 1.5초 |
| LCP | < 2.5초 |
| CLS | < 0.1 |
| JS 번들 (gzipped) | < 150KB |
| TTI | < 3초 |

최적화:
- 코드 스플리팅 (페이지별 dynamic import, Paddle.js 결제 탭에서만)
- SWR 캐시 (staleWhileRevalidate)
- next/font 로컬 호스팅 (Inter, latin + korean subset)
- 이미지: next/image, 로고 SVG 인라인

---

## 15. Phase 0 개발 체크리스트

```
프로젝트 셋업:
  [ ] Next.js 14 + App Router + TypeScript
  [ ] Tailwind CSS
  [ ] Supabase 프로젝트 + DB 스키마 (toc_enabled 포함)
  [ ] Cloudflare Workers 빌드 서버
  [ ] Cloudflare Pages Pro ($20/월)
  [ ] 환경변수 (.env.local)
  [ ] Vercel 배포

인증:
  [ ] /login (Magic Link + Google)
  [ ] /signup (회원가입 + 7일 trial)
  [ ] middleware.ts (라우팅 가드)
  [ ] useUser 훅

온보딩 (2단계):
  [ ] StepProgress (2단계)
  [ ] Step 1: Notion OAuth + DB 선택 + 속성 검증 (통합)
  [ ] Step 2: 서브도메인 + DNS + TOC 설정 + 사이트 생성

레이아웃:
  [ ] DashboardLayout
  [ ] Sidebar (반응형 3단계)
  [ ] AlertBanner (6개 우선순위, D-3/D-1 푸시)
  [ ] DeployFAB (3개 상태: idle/building/done)
  [ ] MobileNav (햄버거 + 오버레이)
  [ ] Toast 시스템

대시보드:
  [ ] BlogSummaryCard (4개 지표)
  [ ] BuildMonitorCard (빌드 성공률, 평균 시간, 서비스 상태)
  [ ] SeoStatusCard (sitemap/RSS 복사)
  [ ] PlanCard (7일 체험, Pro만)
  [ ] 빌드 에러 모달

포스트:
  [ ] FilterTabs
  [ ] PostTable + PostRow
  [ ] SeoIndicator
  [ ] 빈 상태 + 페이지네이션

설정 (2개 탭):
  [ ] SettingsTabs (URL query)
  [ ] GeneralTab (이름, 도메인, 노션, SEO, TOC)
  [ ] BillingTab (체험/Pro/취소/환불)
  [ ] Paddle.js checkout overlay (Pro만)
  [ ] 취소/환불 모달 (멱등성)

FAQ + 지원:
  [ ] /faq 페이지
  [ ] FAQ 아코디언 (5개 항목)
  [ ] 이메일 문의 폼
  [ ] Resend 연동 (관리자 알림)

모니터링:
  [ ] UptimeRobot 상태 페이지 연동
  [ ] 빌드 실패율 계산
  [ ] 평균 빌드 시간 계산
  [ ] Notion 토큰 revoke 시 자동 이메일

공통:
  [ ] SWR 훅 (useSite, usePosts, useBuild)
  [ ] 빌드 폴링 훅 (Workers 상태 조회)
  [ ] 에러 핸들링 (전역 + 컴포넌트)
  [ ] 로딩 스켈레톤
  [ ] 반응형 테스트
```

---

---

## 16. 기술 스펙 요약 (프론트엔드)

> noble-prd-v2.md 섹션 16 참조. 프론트엔드 관련 주요 확정 사항:

| 항목 | 확정 스펙 |
|------|----------|
| SWR 캐시 | /api/site: 30초, /api/posts: 60초, /api/build/status: 폴링 |
| 빌드 폴링 | 0-30초: 2초, 30-120초: 5초, 120-300초: 10초, 300초+: 중단 |
| 일일 배포 리셋 | UTC 00:00 기준 |
| Rate Limiting | 인증 유저 60 req/분, 비인증 IP 20 req/분 |
| 에러 메시지 | 구체적 원인 + 해결책 (한국어) |
| 멱등성 | FAB 클릭 즉시 disabled + Idempotency-Key 헤더 |
| OAuth 쿠키 | HttpOnly, Secure, SameSite=Lax, Max-Age=3600 |
| 첫 빌드 | 온보딩 완료 시 자동 트리거 |

---

*Noble Dashboard PRD v2.2 — 2025-02-13*
*화면 5개, 클릭 최소, 배포는 언제나 1클릭*

**v2.2 변경 사항:**
- 40개 기술 스펙 확정 반영
- SWR 캐시 전략 엔드포인트별 정의
- Rate Limiting 정책 추가
- OAuth 쿠키 설정 명시
- 빌드 폴링 전략 상세화

**v2.1 변경 사항:**
- 온보딩 4단계 → 2단계
- 설정 탭 4개 → 2개 (일반/결제)
- FAB 상태 5개 → 3개 (idle/building/done)
- 체험 14일 → 7일 (D-3, D-1 푸시 강화)
- Phase 0: Pro 플랜만
- 빌드 모니터링 카드 추가
- FAQ + 이메일 문의 폼 추가
- TOC 목차 유저 선택 사항
