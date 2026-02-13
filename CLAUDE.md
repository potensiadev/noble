# CLAUDE.md — Noble (노블) 프로젝트 컨텍스트

> **이 파일은 Claude Code가 매 세션마다 읽는 프로젝트 마스터 문서입니다.**
> **수정 시 반드시 실제 코드 상태와 동기화하세요.**
> **마지막 업데이트: 2025-02-13**

---

## 🎯 프로젝트 한줄 요약

**Noble = 노션 DB → SEO 최적화 정적 블로그 자동 빌드/배포 SaaS**
"노션으로 글만 쓰면, 구글에 잘 잡히는 빠른 블로그가 만들어진다"

---

## 👤 개발자 프로필 (중요!)

- **1인 사업가** — 비개발자, Claude Code + AI로 개발/운영
- **Ship Fast 전략** — 수십개 서비스 런칭 예정, 과도한 엔지니어링 금지
- **Phase 0 MVP** — 5일 내 런칭 목표, 완벽보다 동작하는 제품 우선
- **의사결정 시** — 기술적 장단점을 쉬운 말로 설명하고, 추천 선택지와 이유를 함께 제시할 것

---

## 🏗️ 기술 스택 (절대 변경 금지)

| 레이어 | 기술 | 버전 | 비고 |
|--------|------|------|------|
| 프레임워크 | Next.js (App Router) | 14+ | Vercel 배포 |
| 언어 | TypeScript | 5.3+ | strict 모드 |
| 스타일링 | Tailwind CSS | 3.4+ | 유틸리티 우선 |
| DB/Auth | Supabase (PostgreSQL + Auth) | - | RLS 필수 |
| 데이터 페칭 | SWR | 2.2+ | React Query 사용 금지 |
| 아이콘 | Lucide React | 0.312+ | 트리 셰이킹 |
| 결제 | Paddle (Billing) | - | MoR 모델 |
| 블로그 호스팅 | Cloudflare Pages | - | Pro $20/월, Direct Upload API |
| 빌드 파이프라인 | Cloudflare Workers | - | 별도 분리, Vercel 타임아웃 회피 |
| 이미지 저장 | Cloudflare R2 | - | S3 호환 API |
| 이미지 처리 | sharp (Lambda 호환) | 0.33+ | @img/sharp-linux-x64 사용 |
| 이메일 | Resend | - | Phase 0 선택사항 |
| CSS 유틸리티 | clsx + tailwind-merge | 2.1+ / 2.2+ | 조건부 클래스 |

### 사용하지 않는 것들
- ❌ React Query (SWR 사용)
- ❌ styled-components, CSS Modules (Tailwind 사용)
- ❌ Prisma (Supabase 클라이언트 직접 사용)
- ❌ NextAuth.js (Supabase Auth 사용)
- ❌ Stripe (Paddle 사용)
- ❌ AWS S3 (Cloudflare R2 사용)

---

## 📁 프로젝트 구조

```
src/
├── app/
│   ├── (auth)/                    # 인증 레이아웃 (사이드바 없음)
│   │   ├── login/page.tsx
│   │   ├── signup/page.tsx
│   │   └── layout.tsx             # 센터 정렬, Noble 로고만
│   │
│   ├── (dashboard)/               # 대시보드 레이아웃 (사이드바 있음)
│   │   ├── dashboard/page.tsx     # 메인 대시보드 + 빌드 모니터링
│   │   ├── posts/page.tsx         # 포스트 관리
│   │   ├── settings/page.tsx      # 설정 (탭: general|billing)
│   │   ├── onboarding/page.tsx    # 2단계 온보딩 위저드
│   │   ├── faq/page.tsx           # FAQ 페이지
│   │   └── layout.tsx             # 사이드바 + 알림 배너 + FAB
│   │
│   ├── api/                       # API Routes
│   │   ├── auth/callback/notion/route.ts
│   │   ├── auth/disconnect-notion/route.ts
│   │   ├── site/route.ts          # GET, POST, PATCH
│   │   ├── site/check-subdomain/route.ts
│   │   ├── site/verify-domain/route.ts
│   │   ├── notion/databases/route.ts
│   │   ├── notion/validate-db/route.ts
│   │   ├── notion/validate-url/route.ts
│   │   ├── build/trigger/route.ts
│   │   ├── build/status/route.ts
│   │   ├── build/history/route.ts
│   │   ├── posts/route.ts
│   │   ├── posts/[id]/route.ts
│   │   ├── paddle/webhook/route.ts
│   │   ├── paddle/checkout/route.ts
│   │   ├── paddle/cancel/route.ts
│   │   └── paddle/refund/route.ts
│   │
│   └── layout.tsx                 # 루트 레이아웃 (fonts, metadata)
│
├── components/
│   ├── layout/                    # 레이아웃 컴포넌트
│   │   ├── Sidebar.tsx            # 반응형 3단계 (240px / 64px / 숨김)
│   │   ├── AlertBanner.tsx        # 우선순위 기반 알림 (5종)
│   │   ├── DeployFAB.tsx          # 배포 FAB (5개 상태)
│   │   └── MobileNav.tsx          # 햄버거 + 오버레이
│   │
│   ├── onboarding/                # 온보딩 위저드 (2단계)
│   │   ├── StepProgress.tsx       # 프로그레스 바 (2단계)
│   │   ├── StepNotionConnect.tsx  # Step 1: Notion OAuth + DB 선택 + 속성 검증
│   │   └── StepUrlSetup.tsx       # Step 2: 서브도메인/커스텀도메인 + TOC 설정
│   │
│   ├── dashboard/                 # 대시보드 카드
│   │   ├── BlogSummaryCard.tsx    # 발행/초안/빌드상태/마지막배포
│   │   ├── SeoStatusCard.tsx      # sitemap/RSS 복사 + 서치콘솔 가이드
│   │   └── PlanCard.tsx           # 체험/유료/만료 5개 변형
│   │
│   ├── posts/                     # 포스트 관리
│   │   ├── PostTable.tsx
│   │   ├── PostRow.tsx
│   │   ├── SeoIndicator.tsx       # ✅/⚠️/❌ (meta, slug, image)
│   │   └── FilterTabs.tsx         # 전체/발행됨/초안
│   │
│   ├── settings/                  # 설정 탭 (2개)
│   │   ├── SettingsTabs.tsx       # URL query 기반 탭 전환
│   │   ├── GeneralTab.tsx         # 이름, 도메인, 노션 연동, SEO, TOC 설정
│   │   └── BillingTab.tsx         # 체험/유료/취소/환불
│   │
│   └── ui/                        # 공통 UI 컴포넌트
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
│   ├── useUser.ts                 # Supabase Auth 상태
│   ├── useSite.ts                 # 사이트 데이터 (SWR)
│   ├── usePosts.ts                # 포스트 목록 (SWR)
│   ├── useBuild.ts                # 빌드 상태 + 폴링
│   ├── useDeployLimit.ts          # 배포 횟수 제한 상태
│   └── useToast.ts                # 토스트 알림 관리
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts              # 브라우저용 클라이언트
│   │   └── server.ts              # 서버용 클라이언트 (Service Role Key)
│   ├── paddle.ts                  # Paddle.js 초기화
│   ├── encryption.ts              # AES-256-GCM (서버 전용)
│   ├── idempotency.ts             # 멱등성 키 유틸리티
│   └── utils.ts                   # 상대 시간, slug 검증 등
│
├── types/
│   ├── database.ts                # Supabase 생성 타입
│   └── api.ts                     # API request/response 타입
│
└── middleware.ts                   # 인증 가드 + 온보딩 리다이렉트
```

---

## 🗄️ 데이터베이스 스키마 (Supabase PostgreSQL)

### 핵심 테이블 6개

```
users          — 사용자 (Supabase Auth 연동, 플랜, Paddle 연동)
sites          — 블로그 사이트 (1유저 1사이트, Notion/CF 연동, SEO 설정)
posts          — 포스트 메타 (빌드 시 Notion에서 캐시, 실시간 아님)
builds         — 빌드 히스토리 (상태, 에러, CF 배포 정보)
idempotency_keys   — POST API 멱등성 보장 (24시간 유효)
processed_webhooks — Paddle 웹훅 중복 방지 (30일 보관)
```

### 테이블별 핵심 컬럼

**users:**
- `plan`: 'trial' | 'pro' (기본값: 'trial') — Phase 0는 Pro만
- `trial_ends_at`: 가입 시 NOW() + 7 days (7일 체험)
- `paddle_subscription_status`: 'active' | 'past_due' | 'paused' | 'canceled' | 'trialing' | NULL
- `onboarding_completed`: BOOLEAN (false → /onboarding 리다이렉트)

**sites:**
- `one_site_per_user` UNIQUE 제약 (Phase 0: 1유저 1사이트)
- `notion_database_id` UNIQUE (중복 연동 방지)
- `notion_token_encrypted`: BYTEA (AES-256-GCM)
- `last_build_status`: 'never' | 'building' | 'success' | 'failed'
- `daily_deploy_count` / `daily_deploy_date`: 일일 배포 제한
- `toc_enabled`: BOOLEAN (기본 true, 목차 자동 생성 여부)

**posts:**
- `unique_post_per_site` UNIQUE (site_id, notion_page_id)
- `seo_has_meta`, `seo_has_slug`, `seo_has_image`: SEO 체크 불리언
- `reading_time_min`: 분당 500자(한글) 기준

**builds:**
- `status`: 'queued' | 'building' | 'deploying' | 'success' | 'failed'
- `error_step`: 'notion_auth' | 'notion_fetch' | 'image_processing' | 'html_generation' | 'cloudflare_deploy' | 'timeout'
- `build_duration_ms`: INTEGER (빌드 소요 시간, 모니터링용)

### RLS 정책 (필수!)
모든 테이블에 Row Level Security 활성화. 자기 데이터만 접근 가능.
- users: `auth.uid() = id`
- sites: `auth.uid() = user_id`
- posts/builds: `site_id IN (SELECT id FROM sites WHERE user_id = auth.uid())`

### 자동 트리거
- `updated_at` 컬럼은 UPDATE 시 자동 갱신 (users, sites, posts)

---

## 🔐 보안 규칙 (반드시 준수)

### Notion 토큰 암호화
```
알고리즘: AES-256-GCM
키: 환경변수 ENCRYPTION_KEY_V1, V2... (버전 관리)
IV: 매 암호화마다 랜덤 12바이트
저장 형식: [IV(12)][TAG(16)][ENCRYPTED_DATA] → BYTEA
버전 관리: sites.encryption_version 컬럼으로 키 버전 추적
파일: lib/encryption.ts
```

### 암호화 키 로테이션 (확정)
```
1. ENCRYPTION_KEY_V2 환경변수 추가
2. 신규 암호화는 V2 사용, 복호화는 버전에 맞게
3. 백그라운드 마이그레이션으로 기존 데이터 V2로 전환
4. 전체 완료 후 V1 키 폐기
```

### 인증
- 모든 API Route에서 `getUser()` 호출 → 미인증 시 401
- Supabase JWT (access_token + refresh_token)
- CSRF: Notion OAuth state 파라미터 + SameSite cookie
- Magic Link 만료: 1시간 (Supabase 기본값)
- Magic Link 클릭 즉시 로그인 (별도 인증 불필요)

### OAuth State 쿠키 (확정)
```
httpOnly: true, secure: true, sameSite: 'lax'
maxAge: 10분
state: crypto.randomBytes(32).toString('hex') (64자)
콜백에서 검증 후 즉시 삭제
```

### Paddle 웹훅 검증
1. `Paddle-Signature` 헤더에서 ts, h1 추출
2. ts가 현재 시간 ±5분 이내 확인 (replay 방지)
3. `HMAC-SHA256(webhook_secret, ts + ":" + raw_body) === h1`
4. `event_id`로 중복 체크 (processed_webhooks 테이블)

---

## 🔄 멱등성 정책 (3중 방어 — 반드시 준수!)

> **원칙: "100번 요청해도 결과는 1번과 같다"**

### Layer 1 — 클라이언트
- 버튼 클릭 즉시 disabled + loading 표시
- 응답 전까지 재클릭 차단

### Layer 2 — Idempotency Key (서버)
- 클라이언트가 UUID v4 생성 → `Idempotency-Key` 헤더 전송
- 서버: idempotency_keys 테이블에서 키 조회
  - 존재 → 캐시된 응답 반환 (로직 재실행 안 함)
  - 미존재 → 로직 실행 + 응답 저장
- 키 유효기간: 24시간
- **캐시 범위 (확정)**:
  - 캐시 O: 200 성공, 409 이미 빌드 중
  - 캐시 X: 429 횟수 초과, 402 체험 만료, 500 에러 (상태 변경 가능)

### Layer 3 — DB 제약 (최종 방어)
- sites: `one_site_per_user` UNIQUE
- sites: `notion_database_id` UNIQUE
- processed_webhooks: `event_id` PRIMARY KEY
- builds: 진행 중 빌드 존재 여부 체크

### 엔드포인트별 적용
| 엔드포인트 | L1 | L2 | L3 | 위험도 |
|-----------|----|----|----|----|
| POST `/api/build/trigger` | ✅ | ✅ Idempotency-Key | ✅ 진행 중 빌드 체크 | 🔴 핵심 |
| POST `/api/site` | ✅ | ✅ user_id 체크 | ✅ UNIQUE 제약 | 🔴 핵심 |
| POST `/api/paddle/refund` | ✅ | ✅ Idempotency-Key | ✅ 환불 상태 체크 | 🔴 돈 관련 |
| POST `/api/paddle/cancel` | ✅ | - | ✅ status 사전 체크 | 🟡 |
| POST `/api/paddle/webhook` | - | - | ✅ event_id 중복 체크 | 🔴 핵심 |
| PATCH `/api/site` | - | - | - | 🟢 본질적 멱등 |

---

## 🛤️ 라우팅 & 미들웨어

### 라우팅 테이블
| 경로 | 인증 | 온보딩 필요 | 레이아웃 |
|------|------|-----------|---------|
| `/login`, `/signup` | ❌ | ❌ | AuthLayout (센터) |
| `/onboarding` | ✅ | ❌ (진행 중) | DashboardLayout (사이드바 숨김) |
| `/dashboard` | ✅ | ✅ | DashboardLayout |
| `/posts` | ✅ | ✅ | DashboardLayout |
| `/settings` | ✅ | ✅ | DashboardLayout |

### middleware.ts 로직 (순서 중요!)
1. 비인증 + 보호된 경로 → `/login` 리다이렉트
2. 인증됨 + 로그인/회원가입 → `/dashboard` 리다이렉트
3. 인증됨 + 온보딩 미완료 + 대시보드 경로 → `/onboarding` 리다이렉트
4. 인증됨 + 온보딩 완료 + `/onboarding` → `/dashboard` 리다이렉트

### 체험 만료 시
- 대시보드/포스트/설정: 접근 허용 (읽기 전용 + 업그레이드 유도)
- 배포 API: 402 `TRIAL_EXPIRED` 반환
- 블로그: 서빙 중지

---

## 🏭 빌드 파이프라인 (핵심 비즈니스 로직)

### ⚠️ 아키텍처 중요 사항
**빌드 파이프라인은 Cloudflare Workers에서 실행** (Vercel 서버리스 타임아웃 회피)
- Vercel 무료: 10초, Pro: 60초 제한 → 빌드 불가능
- Cloudflare Workers: CPU 30초, Wall 시간 제한 없음
- 대안: Railway ($5/월) 또는 Render

### 전체 흐름
```
[트리거 (Vercel API)] → [Workers 호출 (HTTP)] → [즉시 202 반환]
                     → [Workers 빌드 실행] → [진행률 콜백 (5회)]
                     → [완료 콜백] → [Supabase 업데이트]
```

### Workers 통신 스펙 (확정)
```
통신 방식: HTTP + 콜백 (D-1)
인증: Bearer Token (D-2)
상태 동기화: Workers → Vercel 콜백 (D-3)

POST https://noble-build.{account}.workers.dev/build
Headers:
  Authorization: Bearer {CF_WORKERS_SECRET}
Body:
  { build_id, site_id, notion_token_encrypted, notion_database_id,
    subdomain, toc_enabled, callback_url }
Response: 202 { accepted: true, build_id }

콜백 (5회): building 시작, fetching 완료, images 완료, deploying 시작, 최종
POST {callback_url}
Headers: Authorization: Bearer {CALLBACK_SECRET}
Body: { build_id, status, step, progress?, error?, duration_ms?, cf_deployment_url? }
콜백 실패 시: 3회 재시도 (1초, 5초, 10초)
```

### 빌드 상태 전이 (확정)
```
queued (30초) → building (4분) → deploying (1분) → success/failed
전체 타임아웃: 5분 (초과 시 failed, error_step: 'timeout')
```

### 빌드 동시 실행 정책
- 진행 중 빌드 있으면 409 반환
- 기존 빌드 완료까지 대기 (취소/큐잉 없음)
- FAB는 building 상태 유지, 기존 빌드 폴링 계속

### Step 1: Notion 데이터 수집
- 토큰 복호화 → Notion API 호출
- `POST /v1/databases/{db_id}/query` (filter: Status = "Published")
- 페이지네이션: `has_more` + `next_cursor`
- Rate limit: **350ms 간격** (Notion 제한: 3 req/sec)
- 각 페이지 블록 재귀 조회: `GET /v1/blocks/{page_id}/children`

### Step 2: 노션 블록 → HTML 변환
주요 매핑:
- paragraph → `<p>`, heading_1/2/3 → `<h1>/<h2>/<h3>` + TOC 앵커
- code → `<pre><code class="language-{lang}">` (Prism.js)
- image → `<figure><img><figcaption>`
- toggle → `<details><summary>`
- callout → `<div class="callout">`
- Rich text: bold→`<strong>`, italic→`<em>`, code→`<code>`, link→`<a>`

### Step 3: 이미지 처리 (CRITICAL!)
> ⚠️ **Notion 이미지 URL은 1시간 후 만료. 반드시 다운로드 필수!**
1. 모든 이미지 URL 수집 (블록 + 커버)
2. 병렬 다운로드 (최대 5개 동시)
3. sharp 라이브러리: WebP 변환, 최대 1200px, 썸네일 400px, EXIF 제거
4. R2 업로드: `{site_id}/images/{content_hash}.webp` (중복 방지)
5. HTML 내 URL → R2 URL 교체
6. `loading="lazy"` + `width/height` 속성 추가 (CLS 방지)

#### 이미지 타임아웃 스펙 (확정)
```
개별 이미지: 다운로드 10초, 처리 5초, 업로드 15초, 최대 10MB
동시 처리: 5개 병렬
전체 이미지 단계: 2분 타임아웃
실패 시: placeholder SVG 삽입 후 빌드 계속 (모든 이미지 실패해도 빌드 성공)
해시: SHA256 앞 16자 (중복 방지, R2 HEAD로 존재 확인)
```

#### R2 접근 정책 (확정)
```
버킷: noble-images (공개 읽기)
URL: https://images.noble.blog/{site_id}/images/{hash}.webp
업로드/삭제: Workers만 (R2 API Key)
사이트 삭제 시: prefix 기반 전체 삭제
```

#### sharp 서버리스 호환성 방안 (3가지)
| 방안 | 설명 | 추천 |
|------|------|------|
| **1. @img/sharp-linux-x64** | Lambda 전용 네이티브 바이너리 패키지 | ✅ 권장 |
| **2. Cloudflare Workers + images.transform** | CF 내장 이미지 변환 API 사용 | Workers 사용 시 |
| **3. 외부 서비스** | Cloudinary/imgix 무료 티어 활용 | 백업 플랜 |

```bash
# 방안 1: Lambda 호환 sharp 설치
npm install @img/sharp-linux-x64 --platform=linux --arch=x64
```

### Step 4: SEO 자동화
- **Slug 자동 생성**: 한글 → transliteration → slugify (최대 60자, 중복 시 -2, -3)
- **Meta tags**: title, description, canonical, OG, Twitter Card
- **OG 이미지**: 커버 없으면 @vercel/og 또는 satori로 자동 생성 (1200×630)
- **sitemap.xml**: 모든 Published 포스트 포함
- **RSS feed**: 최신 20개 포스트, RSS 2.0
- **robots.txt**: User-agent: * / Allow: / / Sitemap URL
- **시맨틱 HTML**: `<article>`, `<nav>`, `<header>`, `<footer>`, `<time>`

### Step 5: TOC (목차) 생성 — 유저 선택 사항
- **설정**: `sites.toc_enabled` (기본값: true)
- toc_enabled = false면 TOC 생성 스킵
- h1/h2/h3 추출 → 중첩 구조 → 3개 미만이면 미생성
- 데스크톱: sticky 우측 사이드바 / 모바일: 접이식

### Step 6: Cloudflare Pages 배포
- Direct Upload API (ZIP → POST)
- 배포 시간 목표: 30초 이내 (50 포스트 기준)
- 빌드 실패 시에도 마지막 성공 배포 유지 (블로그 항상 접속 가능)

### 빌드 에러 처리
| error_step | 사용자 메시지 | 복구 |
|-----------|-------------|------|
| notion_auth | "노션 재연동이 필요합니다" | 재연동 버튼 |
| notion_fetch | "노션에서 데이터를 가져오지 못했습니다" | 자동 재시도 3회 |
| image_processing | "일부 이미지 처리에 실패했습니다" | 스킵 + placeholder |
| html_generation | "HTML 생성 중 오류" | 노션 글 내용 확인 안내 |
| cloudflare_deploy | "Cloudflare 배포 실패" | 자동 재시도 2회 |
| timeout | "빌드 시간 초과 (5분)" | 포스트 수 줄이기 안내 |

---

## 🖥️ 프론트엔드 핵심 컴포넌트

### 배포 FAB (DeployFAB) — 3개 상태
| 상태 | 아이콘 | 텍스트 | 배경 | 클릭 |
|------|-------|-------|------|------|
| idle | Rocket | "배포" | blue-600 | 빌드 트리거 |
| building | Spinner | "배포 중..." | gray-400 | disabled |
| done | Check/X | "완료!"/"실패" | green/red-600 | 3초→idle (실패 시 에러 모달) |

> 기존 5개 상태에서 3개로 간소화. limited/expired는 클릭 시 토스트로 안내.

### 빌드 폴링 전략
```
0~30초:   2초 간격
30~120초: 5초 간격
120~300초: 10초 간격
300초+:   폴링 중단 + "빌드에 시간이 걸리고 있습니다" 표시
```

### 알림 배너 (AlertBanner) — 우선순위 (1개만 표시)
1. 🔴 빌드 실패 → "마지막 배포가 실패했습니다" + [상세 보기]
2. 🟡 Notion 401 → "노션 연동이 해제되었습니다" + [재연동] + 자동 이메일 발송
3. 🟡 결제 실패(past_due) → "결제에 실패했습니다" + [결제 수단 확인]
4. 🟠 체험 D-3 → "체험 기간이 3일 남았습니다" + [업그레이드] + **푸시 이메일**
5. 🟠 체험 D-1 → "체험 기간이 내일 만료됩니다" + [업그레이드] + **푸시 이메일**
6. 🔴 체험 만료 → "체험 기간이 만료되었습니다" + [업그레이드]

> 7일 체험 + D-3, D-1 푸시 강화

### 토스트 시스템
- 위치: 우상단 (top-6 right-6), z-50
- 최대 동시 3개, 초과 시 가장 오래된 것 제거
- 스타일: border-l-4, success(green)/error(red)/warning(yellow)/info(blue)

### 사이드바 반응형
| 브레이크포인트 | 사이드바 | 동작 |
|-------------|---------|------|
| 1024px+ | 240px 고정 | 아이콘 + 텍스트 |
| 768~1023px | 64px 고정 | 아이콘만, 호버 시 툴팁 |
| ~767px | 숨김 | 햄버거 → 오버레이 슬라이드 (280px) |

---

## 💳 결제 시스템 (Paddle)

### 플랜 & 가격 (Phase 0: Pro만)
| 플랜 | 월간 | 연간 (20% 할인) | 일일 배포 |
|------|------|---------------|----------|
| Trial | 무료 (**7일**) | - | 3회 |
| Pro | $9/월 | $86/년 | 10회 |

> Business 플랜은 Phase 1에서 수익화 기능과 함께 출시

### Paddle 연동 방식
- **Checkout**: Paddle.js overlay (인라인 아님)
- **웹훅 이벤트**: subscription.activated, .updated, .canceled, .past_due, transaction.completed, adjustment.created
- **환불**: 14일 이내 자동 전액, 14일 초과 수동 검토
- **구독 취소**: current_period_ends_at까지 서비스 유지, 30일 데이터 보관

### 결제 상태 정책 (확정)
```
Past Due 유예 기간: 7일 (Paddle 재시도 주기와 일치)
- past_due 발생 후 7일간 서비스 유지
- 7일 후 미결제 시 배포 차단

환불 후 데이터: 30일 보관 후 삭제
- 사용자에게 "30일 후 데이터 삭제" 명확히 안내 (이메일 + 대시보드)
- 재구독 시 복원 가능

Trial 만료 후 블로그: 유지 (배포만 차단)
- 마지막 성공 빌드 그대로 서빙
- 새 배포만 402 TRIAL_EXPIRED로 차단
```

### 구독 상태 전이 (확정)
```
NULL(trial) → active (subscription.activated)
active → past_due (subscription.past_due)
past_due → active (결제 수단 변경 성공)
past_due → canceled (7일 후 미결제)
active → canceled (subscription.canceled)
canceled → active (재구독)
```

### 체험 기간 로직 (7일 + 푸시 강화)
```
시작: 가입 시 trial_ends_at = NOW() + 7 days
기능: Pro 전체 기능 (일 3회 배포 제한)
만료 체크: POST /api/build/trigger만 402 반환 (GET/PATCH는 허용)
만료 기준: trial_ends_at 당일 23:59:59 UTC까지 유효
만료 후: 블로그 유지 (배포만 차단), 대시보드 접근 가능
데이터 보관: 30일 → 이후 삭제
알림 (강화):
  - D-3: 배너 + 이메일 (강조)
  - D-1: 배너 + 이메일 (긴급)
  - D+0: 이메일 (만료 안내 + 업그레이드 링크)

일일 배포 카운트 리셋: UTC 00:00 (한국 시간 오전 9시)
```

---

## 🔗 Notion 연동

### OAuth 플로우
```
1. "노션 연동하기" 클릭 → Notion OAuth URL로 이동
   - client_id, response_type=code, owner=user, redirect_uri, state(CSRF)
2. 사용자: Notion에서 DB 선택 + 승인
3. Notion → /api/auth/callback/notion?code={code}&state={state}
4. 서버: state 검증 → token 교환 → AES-256-GCM 암호화 → 저장
5. /onboarding?step=2 리다이렉트
```

### URL 폴백
- "연동이 안 되시나요?" 접힘/펼침 토글
- 정규식: `/([a-f0-9]{32})/` (32자 hex, `?v=` 이전까지)
- Page vs DB 구분: Notion API `object` 필드 체크

### 필수 DB 속성
| 속성 | 타입 | 검사 |
|------|------|------|
| Title | title | 항상 존재 (✅) |
| Slug | rich_text | 이름 정확 일치 "Slug" |
| Status | select | 이름 정확 일치 "Status" + "Published" 옵션 존재 |

### API 호출 패턴
- 기본 간격: 350ms/요청
- 429 응답 시: Retry-After 헤더 우선, 없으면 Exponential backoff (700ms → 1400ms → 2800ms), 최대 3회
- 전체 빌드 타임아웃: 5분
- 401 → 토큰 만료: 사이트 플래그 + 대시보드 배너 + 자동 이메일

### DB 변경 시 데이터 처리 (확정)
```
설정에서 "다른 DB 선택" 시:
1. 확인 모달: "기존 포스트가 삭제됩니다"
2. 사용자 동의 후 posts 삭제 + 새 DB로 빌드
3. 내부적으로 deleted_posts 테이블에 30일 백업 (CS 복구용)
```

### Notion 페이지 삭제 감지 (확정)
```
빌드 시 자동 동기화:
- Notion에 없는 페이지 → posts에서 삭제 + 블로그에서 제거
- 삭제된 포스트 정보(title, slug, deleted_at)를 deleted_posts_log에 보관
- CS 또는 디버깅 시 추적 가능
```

### Notion API 의존 리스크 대응 (3가지)
| 상황 | 감지 | 대응 |
|------|------|------|
| **토큰 revoke** | API 401 | 배너 + **자동 이메일 알림** (Resend) |
| **Notion 장애** | timeout/5xx | **상태 페이지** (UptimeRobot 무료) 연동 |
| **API 사용량 제한** | 429 빈발 | 워크스페이스당 빌드 큐잉 + 지연 안내 |

---

## 🌐 커스텀 도메인

### 서브도메인 규칙
- 형식: `{subdomain}.noble.blog`
- 3~30자, 영문 소문자+숫자+하이픈, 시작/끝 하이픈 불가
- 예약어: admin, api, www, app, dashboard, blog, help, support, status, mail, noble
- 검증: 클라이언트 regex + 서버 DB unique (debounce 500ms)

### 커스텀 도메인 플로우
1. 도메인 입력 → CNAME 안내 (Target: noble-cdn.pages.dev)
2. [DNS 확인하기] → DNS lookup → Cloudflare Pages 등록 → SSL 자동 발급
3. 상태: ✅ 확인됨·SSL 활성 | ⏳ SSL 발급 중 | ❌ CNAME 미발견

---

## 🎨 UI/UX 규칙

### 설계 원칙 (5개)
1. **화면 5개 이하** — 1인 개발 5일 내 구현
2. **배포는 항상 1클릭** — FAB 버튼 모든 화면 상주
3. **Phase 0는 기능만** — 디자인 다듬기는 Phase 1
4. **노션이 에디터** — 대시보드는 관리/모니터링 전용, 글 편집 없음
5. **에러는 구체적으로** — "문제가 발생했습니다" 금지. 항상 원인 + 해결책

### 스타일 규칙
```
알림 배너:
  error:   bg-red-50    border-l-4 border-red-500    text-red-800
  warning: bg-yellow-50 border-l-4 border-yellow-500 text-yellow-800
  info:    bg-blue-50   border-l-4 border-blue-500   text-blue-800

토스트:
  success: bg-green-50 border-l-4 border-green-500 text-green-800
  error:   bg-red-50   border-l-4 border-red-500   text-red-800
  warning: bg-yellow-50 border-l-4 border-yellow-500 text-yellow-800
  info:    bg-blue-50  border-l-4 border-blue-500  text-blue-800

포커스: focus-visible:ring-2 ring-blue-500 ring-offset-2
터치: 모든 인터랙티브 요소 최소 44×44px
```

### 로딩 상태
- 페이지: 전체 스켈레톤 (pulse 1.5s)
- 카드: 카드 높이 회색 블록
- 테이블: 3줄 행 스켈레톤

### 에러 처리 (전역)
```
401 → 로그인 리다이렉트
402 → "업그레이드가 필요합니다" 토스트
429 → "잠시 후 다시 시도해주세요" 토스트
기타 → 구체적 에러 메시지 토스트
```

---

## 📊 성능 목표

| 지표 | 목표 |
|------|------|
| FCP | < 1.5초 |
| LCP | < 2.5초 |
| CLS | < 0.1 |
| JS 번들 (gzipped) | < 150KB |
| TTI | < 3초 |
| Lighthouse (블로그) | 90+ |
| 빌드 시간 (50포스트) | < 30초 |

### 최적화 전략
- 코드 스플리팅: 페이지별 dynamic import
- Paddle.js: 결제 탭에서만 로드
- SWR 캐시: staleWhileRevalidate
- 폰트: next/font 로컬 호스팅 (Inter, latin + korean subset)
- 이미지: next/image, 로고 SVG 인라인

---

## 🔧 환경 변수 (.env.local)

```env
# === Supabase ===
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# === Notion ===
NOTION_CLIENT_ID=
NOTION_CLIENT_SECRET=
NOTION_REDIRECT_URI=

# === 암호화 (버전 관리) ===
ENCRYPTION_KEY_V1=                 # 64자 hex (32바이트 AES-256 키)
ENCRYPTION_KEY_V2=                 # 로테이션 시 추가

# === Cloudflare ===
CLOUDFLARE_ACCOUNT_ID=
CLOUDFLARE_API_TOKEN=
CLOUDFLARE_R2_ACCESS_KEY_ID=
CLOUDFLARE_R2_SECRET_ACCESS_KEY=
CLOUDFLARE_R2_BUCKET=

# === Paddle (Phase 0: Pro만) ===
PADDLE_API_KEY=
PADDLE_WEBHOOK_SECRET=
PADDLE_CLIENT_TOKEN=               # 프론트엔드 Paddle.js용
PADDLE_PRO_MONTHLY_PRICE_ID=
PADDLE_PRO_ANNUAL_PRICE_ID=

# === 빌드 서버 (Cloudflare Workers) ===
CF_WORKERS_URL=                    # 빌드 파이프라인 Workers URL
CF_WORKERS_SECRET=                 # Workers 호출 인증 (Bearer Token)
CALLBACK_SECRET=                   # Workers → Vercel 콜백 인증

# === 기타 ===
NEXT_PUBLIC_BASE_URL=              # https://app.noble.blog
SENTRY_DSN=                        # 에러 트래킹 (선택)
RESEND_API_KEY=                    # 이메일 발송 (필수: 토큰 revoke 알림)
UPTIMEROBOT_API_KEY=               # 상태 페이지 연동 (선택)
```

---

## 📋 Phase 0 개발 순서 (5일 계획)

### Day 1: 프로젝트 셋업 + Notion OAuth
- [ ] Next.js 14 + App Router + TypeScript + Tailwind 초기화
- [ ] Supabase 프로젝트 생성 + DB 스키마 실행 (toc_enabled 포함)
- [ ] 환경변수 설정 (.env.local)
- [ ] middleware.ts (인증 가드)
- [ ] /login, /signup (Magic Link + Google OAuth)
- [ ] useUser 훅
- [ ] Notion OAuth + 토큰 암호화 저장
- [ ] **온보딩 2단계**: Step 1 (Notion 연동 + DB 선택 + 속성 검증)

### Day 2: Notion → 정적 HTML 빌드 파이프라인
- [ ] **Cloudflare Workers 빌드 서버 구축** (Vercel 타임아웃 회피)
- [ ] Notion API → 블록 수집 (페이지네이션, rate limit)
- [ ] 노션 블록 → HTML 변환 (전체 블록 타입)
- [ ] 이미지 다운로드 → WebP 변환 (sharp @img/sharp-linux-x64) → R2 업로드
- [ ] 정적 파일 생성 (index.html, {slug}/index.html)
- [ ] **온보딩 2단계**: Step 2 (URL 설정 + TOC 설정)

### Day 3: SEO 자동화 + Cloudflare 배포
- [ ] Slug 자동 생성 (한글 transliteration)
- [ ] Meta tags, canonical, OG tags 삽입
- [ ] sitemap.xml, rss.xml, robots.txt 생성
- [ ] OG 이미지 자동 생성
- [ ] TOC (목차) 생성 — **toc_enabled 설정 반영**
- [ ] Cloudflare Pages Pro ($20/월) Direct Upload 배포
- [ ] 커스텀 도메인 + SSL

### Day 4: 대시보드 UI
- [ ] DashboardLayout (사이드바 + 알림 배너)
- [ ] DeployFAB (**3개 상태**: idle/building/done + 폴링)
- [ ] MobileNav (햄버거 + 오버레이)
- [ ] /dashboard (BlogSummaryCard + SeoStatusCard + PlanCard + **빌드 모니터링**)
- [ ] /posts (FilterTabs + PostTable + SeoIndicator)
- [ ] /settings (**2개 탭**: 일반/결제)
- [ ] **/faq 페이지 + 이메일 문의 폼**
- [ ] Toast 시스템
- [ ] 빌드 에러 모달

### Day 5: 결제 + 모니터링 + QA + 런칭
- [ ] Paddle.js checkout overlay (**Pro만**)
- [ ] Paddle 웹훅 수신 + 처리
- [ ] 구독 취소/환불 모달 (멱등성 3중 방어)
- [ ] 체험 만료 로직 (**7일** + D-3, D-1 푸시)
- [ ] 배포 횟수 제한
- [ ] **빌드 모니터링 대시보드** (실패율, 평균 빌드 시간)
- [ ] **UptimeRobot 상태 페이지 연동**
- [ ] **Notion 토큰 revoke 시 자동 이메일**
- [ ] 에러 핸들링 (전역 + 컴포넌트)
- [ ] 로딩 스켈레톤
- [ ] 반응형 테스트
- [ ] Vercel 배포
- [ ] QA

---

## ⚠️ 코딩 규칙 (Claude Code 필독!)

### 반드시 지킬 것
1. **TypeScript strict 모드** — any 타입 사용 금지
2. **서버 컴포넌트 우선** — 클라이언트 컴포넌트는 'use client' 명시적 선언
3. **SWR 사용** — 서버 데이터 페칭은 반드시 SWR 훅으로
4. **RLS 의존** — 서버에서 직접 SQL 작성 시에도 RLS 활성화 확인
5. **에러 메시지 구체적으로** — 사용자에게 원인 + 해결책 제시
6. **멱등성 보장** — POST 엔드포인트에서 반드시 3중 방어 적용
7. **Notion 이미지 반드시 다운로드** — URL 직접 사용 절대 금지
8. **Notion API 350ms 간격** — rate limit 준수
9. **한국어 UI** — 모든 사용자 메시지는 한국어
10. **Tailwind 클래스 순서** — clsx + tailwind-merge 사용

### 하지 말 것
1. ❌ 과도한 추상화 — 1인 개발, 간결하게
2. ❌ 글로벌 상태 관리 라이브러리 (Redux, Zustand 등) — SWR + React state 충분
3. ❌ 테스트 코드 (Phase 0) — 수동 QA로 충분
4. ❌ CI/CD 파이프라인 구축 — Vercel 자동 배포 사용
5. ❌ 마이크로서비스 — 모놀리스 (Next.js API Routes)
6. ❌ Docker — Vercel 서버리스
7. ❌ ORM (Prisma 등) — Supabase 클라이언트 직접 사용
8. ❌ 국제화 (i18n) — Phase 0는 한국어만

### 파일 작성 컨벤션
- 컴포넌트: PascalCase (`BlogSummaryCard.tsx`)
- 훅: camelCase with use prefix (`useUser.ts`)
- 유틸리티: camelCase (`encryption.ts`)
- 타입: PascalCase (`Database`, `ApiResponse`)
- API Route: `route.ts` (Next.js App Router 규칙)

### 커밋 메시지
```
feat: 기능 추가
fix: 버그 수정
refactor: 리팩토링
style: UI/스타일 변경
docs: 문서 수정
```

---

## 📌 자주 참조하는 PRD 섹션

상세 스펙이 필요하면 프로젝트 루트의 아래 파일 참조:
- **noble-prd-v2.md** — 전체 제품 요구사항 (기술 아키텍처, API 설계, 빌드 파이프라인, 결제, 보안)
- **noble-dashboard-prd-v2.md** — 프론트엔드 상세 설계 (화면별 레이아웃, 컴포넌트 상세, 상태 머신, 에러 처리)

---

## 🚀 현재 상태

- **Phase**: Phase 0 MVP
- **진행 상황**: 기획 완료, **모든 기술 스펙 확정**, 개발 시작 전
- **다음 단계**: Day 1 — 프로젝트 셋업 + Notion OAuth
- **주요 변경 (v2.2)**:
  - 온보딩 4단계 → 2단계
  - 체험 14일 → 7일 (D-3, D-1 푸시 강화)
  - 설정 탭 4개 → 2개 (일반/결제)
  - FAB 상태 5개 → 3개
  - 빌드 파이프라인 Cloudflare Workers 분리
  - Pro 플랜만 런칭 (Business는 Phase 1)
  - 빌드 모니터링 + FAQ 추가
  - **40개 기술 스펙 확정 (17개 의사결정 + 23개 상세 정의)**

---

## 📐 상세 기술 스펙 요약 (v2.2 확정)

### 의사결정 요약 (17개)
| # | 항목 | 결정 |
|---|------|------|
| D-1 | Workers 통신 | HTTP + 콜백 |
| D-2 | Workers 인증 | Bearer Token |
| D-3 | 상태 동기화 | Workers → Vercel 콜백 |
| D-4 | Trial 만료 블로그 | 유지 (배포만 차단) |
| D-5 | Past Due 유예 | 7일 |
| D-6 | 환불 후 데이터 | 30일 보관 + 안내 |
| D-7 | DB 변경 시 | 확인 모달 + 내부 백업 |
| D-8 | 페이지 삭제 | 자동 동기화 + 로그 |
| D-9 | 이미지 실패 | 모두 실패해도 빌드 성공 |
| D-10 | 빌드 동시 실행 | 기존 빌드 대기 |
| D-11 | CF Pages 생성 | 온보딩 완료 시 |
| D-12 | Magic Link 만료 | 1시간 |
| D-13 | 이메일 인증 | 불필요 |
| D-14 | FAQ 발신자 | Reply-To 헤더 |
| D-15 | 모니터링 대상 | Notion + CF |
| D-16 | 빌드 히스토리 | 90일 + 요약 |
| D-17 | Sentry 범위 | Phase 0 서버만 |

### Rate Limiting (확정)
```
/api/build/trigger: 20회/시간 (user)
/api/site POST: 5회/시간 (user)
/api/site/check-subdomain: 30회/분 (ip)
/api/auth/*: 10회/15분 (ip)
/api/support/contact: 3회/시간 (user)
```

### SWR 캐시 전략 (확정)
```
/api/site: 30초 갱신
/api/posts: 1분 갱신
/api/build/status: 수동 폴링
에러 시: 401/403 재시도 안함, 5xx 캐시 유지 + 재시도
```

### 토스트 큐 (확정)
```
최대 3개, FIFO (오래된 것 제거)
dedupKey로 동일 토스트 중복 방지
error는 duration: 0 (수동 닫기)
```

### OG 이미지 템플릿 (확정)
```
크기: 1200×630px
배경: 블루-바이올렛 그라데이션 (#1E40AF → #7C3AED)
폰트: Pretendard (한글 지원)
구성: 블로그 이름 + 포스트 제목 + noble.blog
생성: @vercel/og, 저장: R2
```

---

*이 파일을 업데이트할 때는 실제 코드와 동기화하세요.*
*마지막 업데이트: 2025-02-13 (v2.2 - 모든 기술 스펙 확정)*
