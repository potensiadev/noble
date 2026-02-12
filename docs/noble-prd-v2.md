# Noble (노블) — Product Requirements Document v2.0

> **Document Status:** Final Draft
> **Last Updated:** 2025-02-13
> **Author:** Noble Product Team
> **Phase:** Phase 0 MVP → Phase 1 → Phase 2

---

## 1. 제품 개요

### 1.1 미션

"노션으로 글만 쓰면, 구글에 잘 잡히는 빠른 블로그가 만들어진다"

### 1.2 포지셔닝

워드프레스의 SEO 파워 + 노션의 편리함, 절반의 가격.

### 1.3 타겟 사용자

블로그로 수익(애드센스, 제휴, 유료 구독)을 내고 싶지만, 워드프레스는 복잡하고 oopy는 SEO가 약한 사용자.

### 1.4 핵심 가치

| 가치 | 설명 | 측정 기준 |
|------|------|----------|
| **속도** | 정적 HTML + CDN. oopy 대비 10배+ 빠름 | Lighthouse 90+, LCP < 1.5s |
| **SEO 자동화** | slug, meta, sitemap, RSS, OG, canonical, 시맨틱 HTML 전자동 | 배포 후 추가 설정 0건 |
| **간편함** | 노션에서 글 쓰고 배포 버튼 1클릭 | 온보딩 완료 3분 이내 |
| **수익화** | 애드센스, 제휴, 멤버십 원클릭 셋업 (Phase 2) | 수익화 설정 5분 이내 |

### 1.5 경쟁 비교

| 기능 | oopy | 워드프레스 | **Noble** |
|------|------|-----------|-----------|
| SEO 자동화 | ❌ 수동 | ⚠️ 플러그인 의존 | ✅ 전자동 |
| 페이지 속도 | ⚠️ 느림 (실시간 프록시) | ⚠️ 호스팅 의존 | ✅ 정적 CDN |
| sitemap/RSS | ❌ 제한적 | ⚠️ 플러그인 | ✅ 자동 생성 |
| 목차(TOC) | ❌ 없음 | ⚠️ 플러그인 | ✅ 자동 생성 |
| 이전/다음글 | ❌ 없음 | ✅ 내장 | ✅ 자동 생성 |
| 카테고리/태그 | ❌ 없음 | ✅ 내장 | ✅ 노션 속성 연동 |
| 구조화 데이터 | ❌ 없음 | ⚠️ 플러그인 | ✅ JSON-LD 자동 |
| 노션 글쓰기 | ✅ | ❌ | ✅ |
| 설정 난이도 | ✅ 쉬움 | ❌ 어려움 | ✅ 3분 셋업 |
| 수익화 도구 | ⚠️ 수동 | ✅ 풍부 | ✅ 원클릭 |
| 가격 | ₩10,890/월 | $20~30/월 | **$9/월** |

---

## 2. 기술 아키텍처

### 2.1 시스템 구성도

```
┌─────────────┐     ┌──────────────┐     ┌──────────────────┐
│   사용자     │     │  Noble App   │     │   Notion API     │
│  (브라우저)  │────▶│  (Next.js)   │────▶│  (Read Only)     │
└─────────────┘     │  Vercel      │     └──────────────────┘
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
     ┌──────────────┐ ┌────────┐ ┌──────────┐
     │  Supabase    │ │ CF R2  │ │ CF Pages │
     │  (DB+Auth)   │ │(이미지)│ │(블로그)  │
     └──────────────┘ └────────┘ └──────────┘
              │
              ▼
     ┌──────────────┐
     │   Paddle     │
     │  (결제/MoR)  │
     └──────────────┘
```

### 2.2 기술 스택

| 레이어 | 기술 | 버전 | 선택 이유 |
|--------|------|------|----------|
| **프론트엔드/API** | Next.js | 14+ (App Router) | Vercel 무료 배포, API Routes 내장, RSC 지원 |
| **호스팅 (대시보드)** | Vercel | Free tier | 자동 배포, 서버리스 함수, 글로벌 CDN |
| **호스팅 (블로그)** | Cloudflare Pages | Free tier | 500빌드/월, 무제한 요청, 커스텀 도메인+SSL 무료 |
| **DB** | Supabase PostgreSQL | Free tier | 500MB DB, 1GB 파일, Auth 내장, RLS 보안 |
| **인증** | Supabase Auth | - | Magic Link + Google OAuth, JWT 자동 관리 |
| **이미지 저장** | Cloudflare R2 | Free tier | 10GB 저장, 10M reads/월, S3 호환 API |
| **결제** | Paddle (Billing) | - | MoR 모델, 글로벌 VAT 자동 처리, 구독 관리 내장 |
| **이메일** | Resend | Free tier | 트랜잭션 이메일(빌드 완료, 체험 만료 알림) |
| **AI (Phase 1+)** | Claude API (Haiku) | - | Meta description 생성 (~$0.01/글) |
| **모니터링** | Sentry (Free) | - | 에러 트래킹, 성능 모니터링 |

### 2.3 인프라 비용 분석

| 시나리오 | 유저 수 | 월 수령액 | 월 인프라 비용 | 월 순이익 |
|---------|---------|----------|-------------|----------|
| 초기 | 50명 | $402 | ~$0 (무료 티어) | **$402** |
| 성장기 | 200명 | $1,610 | ~$25 (Supabase Pro) | **$1,585** |
| 안정기 | 500명 | $4,025 | ~$65 | **$3,960** |
| 스케일 | 1,000명 | $8,050 | ~$120 | **$7,930** |

무료 티어 한계 도달 시점:
- Supabase: ~500명 (DB 500MB) → Pro $25/월
- Cloudflare Pages: 500빌드/월 → ~50명(일 배포 평균 기준) 이후 Pro $20/월
- Vercel: 100GB 대역폭 → ~1,000명 이후 Pro $20/월

---

## 3. 데이터 모델

### 3.1 데이터베이스 스키마

```sql
-- ===================================
-- 1. Users (Supabase Auth 연동)
-- ===================================
CREATE TABLE public.users (
  id              UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email           TEXT NOT NULL,
  display_name    TEXT,
  avatar_url      TEXT,
  
  -- 플랜 정보
  plan            TEXT NOT NULL DEFAULT 'trial'
                  CHECK (plan IN ('trial', 'pro', 'business')),
  trial_ends_at   TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '14 days'),
  
  -- Paddle 연동
  paddle_customer_id    TEXT,
  paddle_subscription_id TEXT,
  paddle_subscription_status TEXT
                  CHECK (paddle_subscription_status IN (
                    'active', 'past_due', 'paused', 'canceled', 'trialing', NULL
                  )),
  current_period_ends_at TIMESTAMPTZ,
  
  -- 상태
  onboarding_completed BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ===================================
-- 2. Sites (블로그 사이트)
-- ===================================
CREATE TABLE public.sites (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
  
  -- 블로그 기본 정보
  name            TEXT NOT NULL DEFAULT 'My Blog',
  description     TEXT DEFAULT '',
  subdomain       TEXT NOT NULL UNIQUE,       -- {subdomain}.noble.blog
  custom_domain   TEXT UNIQUE,                -- blog.example.com (nullable)
  
  -- Notion 연동
  notion_token_encrypted  BYTEA NOT NULL,     -- AES-256-GCM 암호화된 access_token
  notion_workspace_id     TEXT NOT NULL,
  notion_workspace_name   TEXT,
  notion_database_id      TEXT NOT NULL UNIQUE, -- 중복 연동 방지
  notion_bot_id           TEXT,
  
  -- Cloudflare 연동
  cf_project_name TEXT,                       -- Cloudflare Pages 프로젝트명
  cf_deployment_url TEXT,                     -- 최신 배포 URL
  
  -- SEO 설정
  default_og_image_url    TEXT,               -- 기본 OG 이미지
  default_meta_description TEXT,              -- 기본 meta description
  robots_allow_crawling   BOOLEAN NOT NULL DEFAULT TRUE,
  
  -- 빌드 상태
  last_build_status TEXT DEFAULT 'never'
                    CHECK (last_build_status IN ('never', 'building', 'success', 'failed')),
  last_build_at     TIMESTAMPTZ,
  last_build_error  TEXT,                     -- 실패 시 에러 메시지
  
  -- 배포 횟수 제한
  daily_deploy_count INTEGER NOT NULL DEFAULT 0,
  daily_deploy_date  DATE NOT NULL DEFAULT CURRENT_DATE,
  
  -- DNS 상태
  custom_domain_verified BOOLEAN DEFAULT FALSE,
  ssl_status            TEXT DEFAULT 'none'
                        CHECK (ssl_status IN ('none', 'pending', 'active', 'error')),
  
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  CONSTRAINT one_site_per_user UNIQUE (user_id)  -- Phase 0: 1유저 1사이트
);

-- ===================================
-- 3. Posts (빌드 시 캐시된 포스트 메타)
-- ===================================
CREATE TABLE public.posts (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  site_id         UUID NOT NULL REFERENCES public.sites(id) ON DELETE CASCADE,
  notion_page_id  TEXT NOT NULL,
  
  -- 포스트 메타 (빌드 시 Notion에서 가져옴)
  title           TEXT NOT NULL,
  slug            TEXT,                       -- nullable: 빈 slug 허용, 빌드 시 자동생성
  status          TEXT NOT NULL DEFAULT 'Draft'
                  CHECK (status IN ('Published', 'Draft')),
  
  -- SEO 메타
  meta_title      TEXT,                       -- 커스텀 meta title (null이면 title 사용)
  meta_description TEXT,                      -- 커스텀 description (null이면 본문 160자)
  og_image_url    TEXT,                       -- 포스트별 OG 이미지
  
  -- 콘텐츠 메타
  word_count      INTEGER DEFAULT 0,
  reading_time_min INTEGER DEFAULT 0,         -- 분당 500자(한글) 기준
  has_cover_image BOOLEAN DEFAULT FALSE,
  headings_count  INTEGER DEFAULT 0,          -- TOC 생성 여부 판단용
  
  -- SEO 체크
  seo_has_meta    BOOLEAN DEFAULT FALSE,
  seo_has_slug    BOOLEAN DEFAULT FALSE,
  seo_has_image   BOOLEAN DEFAULT FALSE,
  
  -- Notion 메타
  notion_last_edited TIMESTAMPTZ,
  notion_url      TEXT,                       -- 노션 원본 링크 (편집용)
  
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  
  CONSTRAINT unique_post_per_site UNIQUE (site_id, notion_page_id)
);

-- ===================================
-- 4. Builds (빌드 히스토리)
-- ===================================
CREATE TABLE public.builds (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  site_id         UUID NOT NULL REFERENCES public.sites(id) ON DELETE CASCADE,
  
  status          TEXT NOT NULL DEFAULT 'queued'
                  CHECK (status IN ('queued', 'building', 'deploying', 'success', 'failed')),
  
  -- 빌드 메타
  total_posts     INTEGER DEFAULT 0,
  published_posts INTEGER DEFAULT 0,
  total_images    INTEGER DEFAULT 0,
  build_duration_ms INTEGER,                  -- 빌드 소요 시간
  
  -- 에러
  error_message   TEXT,
  error_step      TEXT,                       -- 어느 단계에서 실패했는지
  
  -- Cloudflare
  cf_deployment_id TEXT,
  cf_deployment_url TEXT,
  
  triggered_by    TEXT DEFAULT 'manual'
                  CHECK (triggered_by IN ('manual', 'webhook', 'system')),
  
  started_at      TIMESTAMPTZ,
  completed_at    TIMESTAMPTZ,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- ===================================
-- 5. Idempotency (멱등성 보장)
-- ===================================

-- Paddle 웹훅 중복 처리 방지
CREATE TABLE public.processed_webhooks (
  event_id        TEXT PRIMARY KEY,              -- Paddle event_id
  event_type      TEXT NOT NULL,                 -- subscription.activated 등
  processed_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- POST API 멱등성 키 (빌드 트리거, 사이트 생성, 환불 등)
CREATE TABLE public.idempotency_keys (
  key             TEXT PRIMARY KEY,              -- 클라이언트 생성 UUID
  user_id         UUID NOT NULL REFERENCES public.users(id) ON DELETE CASCADE,
  endpoint        TEXT NOT NULL,                 -- /api/build/trigger 등
  response_status INTEGER NOT NULL,              -- 200, 409 등
  response_body   JSONB NOT NULL,                -- 캐시된 응답
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 24시간 지난 멱등성 키 자동 정리 (Supabase pg_cron 또는 앱 레벨)
-- DELETE FROM public.idempotency_keys WHERE created_at < NOW() - INTERVAL '24 hours';
-- DELETE FROM public.processed_webhooks WHERE processed_at < NOW() - INTERVAL '30 days';

-- ===================================
-- 6. Indexes
-- ===================================
CREATE INDEX idx_sites_user_id ON public.sites(user_id);
CREATE INDEX idx_sites_subdomain ON public.sites(subdomain);
CREATE INDEX idx_sites_custom_domain ON public.sites(custom_domain) WHERE custom_domain IS NOT NULL;
CREATE INDEX idx_sites_notion_db ON public.sites(notion_database_id);
CREATE INDEX idx_posts_site_id ON public.posts(site_id);
CREATE INDEX idx_posts_status ON public.posts(site_id, status);
CREATE INDEX idx_builds_site_id ON public.builds(site_id);
CREATE INDEX idx_builds_status ON public.builds(site_id, status);

-- ===================================
-- 6. Row Level Security
-- ===================================
ALTER TABLE public.users ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.sites ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.builds ENABLE ROW LEVEL SECURITY;

-- Users: 자기 데이터만 읽기/수정
CREATE POLICY users_self ON public.users
  FOR ALL USING (auth.uid() = id);

-- Sites: 자기 사이트만 접근
CREATE POLICY sites_owner ON public.sites
  FOR ALL USING (auth.uid() = user_id);

-- Posts: 자기 사이트의 포스트만 접근
CREATE POLICY posts_owner ON public.posts
  FOR ALL USING (
    site_id IN (SELECT id FROM public.sites WHERE user_id = auth.uid())
  );

-- Builds: 자기 사이트의 빌드만 접근
CREATE POLICY builds_owner ON public.builds
  FOR ALL USING (
    site_id IN (SELECT id FROM public.sites WHERE user_id = auth.uid())
  );

-- ===================================
-- 7. 자동 업데이트 트리거
-- ===================================
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at BEFORE UPDATE ON public.users
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER sites_updated_at BEFORE UPDATE ON public.sites
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
CREATE TRIGGER posts_updated_at BEFORE UPDATE ON public.posts
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

### 3.2 Notion 토큰 암호화

```
알고리즘: AES-256-GCM
키: 환경변수 ENCRYPTION_KEY (32바이트)
IV: 매 암호화마다 랜덤 12바이트 생성
저장 형식: [IV(12bytes)][TAG(16bytes)][ENCRYPTED_DATA]
```

암호화/복호화 유틸:
```typescript
// lib/encryption.ts
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const KEY = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex'); // 64자 hex = 32bytes

export function encrypt(text: string): Buffer {
  const iv = randomBytes(12);
  const cipher = createCipheriv(ALGORITHM, KEY, iv);
  const encrypted = Buffer.concat([cipher.update(text, 'utf8'), cipher.final()]);
  const tag = cipher.getAuthTag();
  return Buffer.concat([iv, tag, encrypted]); // 12 + 16 + N bytes
}

export function decrypt(data: Buffer): string {
  const iv = data.subarray(0, 12);
  const tag = data.subarray(12, 28);
  const encrypted = data.subarray(28);
  const decipher = createDecipheriv(ALGORITHM, KEY, iv);
  decipher.setAuthTag(tag);
  return decipher.update(encrypted) + decipher.final('utf8');
}
```

---

## 4. API 설계

### 4.1 API 엔드포인트 목록

모든 API는 Next.js App Router의 Route Handlers로 구현. 인증 필요 엔드포인트는 Supabase JWT 검증.

#### 인증 (Auth)

| Method | Endpoint | 인증 | 설명 |
|--------|----------|------|------|
| GET | `/api/auth/callback/notion` | ✅ | Notion OAuth 콜백 처리 |
| POST | `/api/auth/disconnect-notion` | ✅ | 노션 연동 해제 |

#### 사이트 (Sites)

| Method | Endpoint | 인증 | 멱등성 | 설명 |
|--------|----------|------|-------|------|
| GET | `/api/site` | ✅ | - (GET) | 내 사이트 정보 조회 |
| POST | `/api/site` | ✅ | ✅ user_id UNIQUE | 사이트 생성 (온보딩 완료 시). user_id당 1개 제약으로 중복 생성 방지 |
| PATCH | `/api/site` | ✅ | ✅ 본질적 멱등 | 사이트 설정 수정 (PUT/PATCH는 동일 값 반복 저장해도 결과 동일) |
| POST | `/api/site/check-subdomain` | ✅ | - (조회) | 서브도메인 사용 가능 여부 체크 |
| POST | `/api/site/verify-domain` | ✅ | - (조회) | 커스텀 도메인 DNS 검증 |

#### 노션 (Notion)

| Method | Endpoint | 인증 | 설명 |
|--------|----------|------|------|
| GET | `/api/notion/databases` | ✅ | 연동된 DB 목록 조회 |
| POST | `/api/notion/validate-db` | ✅ | DB 필수 속성 검사 |
| POST | `/api/notion/validate-url` | ✅ | URL 폴백: URL에서 ID 추출 + 검증 |

#### 빌드/배포 (Builds)

| Method | Endpoint | 인증 | 멱등성 | 설명 |
|--------|----------|------|-------|------|
| POST | `/api/build/trigger` | ✅ | ✅ Idempotency-Key | 빌드 트리거 (배포 버튼) |
| GET | `/api/build/status` | ✅ | - (GET) | 현재 빌드 상태 조회 |
| GET | `/api/build/history` | ✅ | - (GET) | 빌드 히스토리 |

#### 포스트 (Posts)

| Method | Endpoint | 인증 | 설명 |
|--------|----------|------|------|
| GET | `/api/posts` | ✅ | 포스트 목록 (캐시된 데이터) |
| GET | `/api/posts/[id]` | ✅ | 포스트 상세 |

#### 결제 (Paddle)

| Method | Endpoint | 인증 | 멱등성 | 설명 |
|--------|----------|------|-------|------|
| POST | `/api/paddle/webhook` | ❌* | ✅ event_id | Paddle 웹훅 수신 (*Paddle 서명 검증) |
| POST | `/api/paddle/checkout` | ✅ | ✅ Paddle 자체 | Paddle Checkout 세션 생성 |
| POST | `/api/paddle/cancel` | ✅ | ✅ 상태 체크 | 구독 취소 요청 |
| POST | `/api/paddle/refund` | ✅ | ✅ Idempotency-Key | 환불 요청 |

### 4.2 주요 API 상세 스펙

#### `POST /api/build/trigger` — 배포 트리거

핵심 비즈니스 로직이 가장 많은 엔드포인트.

```
Request:
  Headers:
    Authorization: Bearer {supabase_jwt}
    Idempotency-Key: {client_generated_uuid}   ← 멱등성 키 (필수)
  Body: (없음)

Response 200:
  { "build_id": "uuid", "status": "queued" }

Response 429:
  { "error": "DEPLOY_LIMIT_REACHED", "message": "오늘 배포 횟수를 초과했습니다.",
    "limit": 3, "used": 3, "resets_at": "2025-02-14T00:00:00Z" }

Response 402:
  { "error": "TRIAL_EXPIRED", "message": "체험 기간이 만료되었습니다." }

Response 409:
  { "error": "BUILD_IN_PROGRESS", "message": "이미 빌드가 진행 중입니다.",
    "existing_build_id": "uuid" }
```

멱등성 보장:
```
클라이언트:
  - FAB 클릭 시 UUID v4 생성 → Idempotency-Key 헤더로 전송
  - 클릭 즉시 FAB disabled (더블클릭 방지 1차 방어)

서버:
  1. Idempotency-Key로 idempotency_keys 테이블 조회
  2. 존재하면 → 캐시된 응답 즉시 반환 (새 빌드 생성 안 함)
  3. 없으면 → 정상 처리 후 응답을 idempotency_keys에 저장
  4. 키 유효기간: 24시간 (이후 자동 삭제)
```

서버 로직:
```
1. JWT에서 user_id 추출
2. Idempotency-Key 체크:
   - idempotency_keys 테이블에 해당 키 존재? → 캐시된 응답 반환 (종료)
3. sites 테이블에서 사이트 조회
4. 사전 검증:
   a. 노션 토큰 유효성 (401 시 → NOTION_DISCONNECTED 에러)
   b. 플랜 확인: trial이면 trial_ends_at 체크
   c. 배포 횟수 확인:
      - daily_deploy_date가 오늘이 아니면 → count 리셋
      - trial: 3회, pro: 10회, business: 무제한
      - 초과 시 429 반환
   d. 진행 중인 빌드 확인 → 409 + existing_build_id 반환
5. builds 레코드 생성 (status: 'queued')
6. daily_deploy_count +1
7. idempotency_keys에 키 + 응답 저장
8. 빌드 작업 비동기 실행 (아래 빌드 파이프라인 참고)
9. build_id 즉시 반환
```

#### `GET /api/build/status` — 빌드 상태 폴링

```
Request:
  Headers: Authorization: Bearer {supabase_jwt}
  Query: ?build_id={uuid}  (optional, 없으면 최신 빌드)

Response 200:
  {
    "build_id": "uuid",
    "status": "building",      // queued | building | deploying | success | failed
    "progress": {
      "step": "fetching_pages", // fetching_pages | converting | generating_seo |
                                 // optimizing_images | deploying | done
      "current": 15,
      "total": 32
    },
    "started_at": "...",
    "duration_ms": 12500,
    "error": null
  }
```

클라이언트 폴링 전략:
```
빌드 시작 후:
  - 0~30초: 2초 간격 폴링
  - 30~120초: 5초 간격
  - 120초+: 10초 간격
  - 300초+: 폴링 중단, "빌드에 시간이 걸리고 있습니다" 표시
```

#### `GET /api/auth/callback/notion` — Notion OAuth 콜백

```
Flow:
  1. Notion OAuth 화면에서 승인
  2. Notion이 ?code={auth_code}&state={csrf_token} 으로 리다이렉트
  3. 서버에서:
     a. state 검증 (CSRF 방지)
     b. 멱등성 체크: 해당 user에게 이미 유효한 notion_token이 있으면
        → 토큰 교환 스킵, /onboarding?step=2 로 바로 리다이렉트
     c. auth_code로 access_token 교환
        POST https://api.notion.com/v1/oauth/token
        Body: { grant_type, code, redirect_uri }
        Auth: Basic base64(client_id:client_secret)
     d. code 재사용 방지: Notion이 이미 사용된 code에 대해 에러 반환
        → 에러 시 "이미 연동이 완료되었습니다" 메시지로 /onboarding?step=2 리다이렉트
     e. 응답에서 추출:
        - access_token
        - workspace_id
        - workspace_name
        - bot_id
     f. access_token을 AES-256-GCM으로 암호화
     g. sites 테이블에 저장 (또는 임시 세션에 저장 → 온보딩 완료 시 sites에 이동)
  4. /onboarding?step=2 로 리다이렉트
```

#### `POST /api/paddle/webhook` — Paddle 웹훅 수신

```
검증:
  - Paddle-Signature 헤더로 웹훅 서명 검증
  - ts 값으로 5분 이내 타임스탬프 확인 (replay 방지)

처리할 이벤트:
  1. subscription.activated
     → users.plan = 매핑(price_id), paddle_subscription_status = 'active'
  
  2. subscription.updated
     → 플랜 변경(upgrade/downgrade) 반영
  
  3. subscription.canceled
     → paddle_subscription_status = 'canceled'
     → current_period_ends_at 까지 서비스 유지
  
  4. subscription.past_due
     → paddle_subscription_status = 'past_due'
     → 대시보드에 결제 실패 배너 표시
  
  5. transaction.completed
     → 결제 성공 로깅
  
  6. adjustment.created (환불)
     → 환불 상태 기록, 자동 환불(14일 이내)이면 구독 즉시 취소

멱등성:
  1. event_id로 processed_webhooks 테이블 조회
  2. 이미 처리됨 → 200 OK 즉시 반환 (재처리 안 함)
  3. 미처리 → 트랜잭션 내에서:
     a. 비즈니스 로직 실행
     b. processed_webhooks에 event_id 삽입
     c. 커밋
  4. 30일 지난 레코드 자동 정리
```

---

## 5. 빌드 파이프라인 상세

### 5.1 전체 빌드 플로우

```
[트리거] → [Notion 데이터 수집] → [마크다운 변환] → [이미지 처리]
        → [HTML 생성] → [SEO 자동화] → [Cloudflare 배포] → [완료]
```

### 5.2 단계별 상세

#### Step 1: Notion 데이터 수집

```
1. 토큰 복호화
2. Notion API 호출:
   POST https://api.notion.com/v1/databases/{db_id}/query
   Body: {
     filter: { property: "Status", select: { equals: "Published" } },
     sorts: [{ property: "Created time", direction: "descending" }]
   }
   
3. 페이지네이션 처리:
   - has_more가 true인 동안 next_cursor로 반복 호출
   - Rate limit: Notion API 3 req/sec → 350ms 간격 호출
   
4. 각 페이지 블록 조회:
   GET https://api.notion.com/v1/blocks/{page_id}/children?page_size=100
   - 재귀적으로 하위 블록 수집 (toggle, column 등)
   
5. 메타데이터 추출:
   - title: Title 속성
   - slug: Slug 속성 (비어있으면 title에서 자동 생성)
   - status: Status 속성
   - cover: 페이지 커버 이미지 URL
   - created_time, last_edited_time
   - 기타 Select/Multi-select 속성 (카테고리/태그용 - Phase 1)
```

#### Step 2: 마크다운 변환

```
노션 블록 타입 → 마크다운/HTML 매핑:

| 노션 블록 | 변환 결과 |
|-----------|----------|
| paragraph | <p> |
| heading_1 | <h1> + TOC 앵커 |
| heading_2 | <h2> + TOC 앵커 |
| heading_3 | <h3> + TOC 앵커 |
| bulleted_list_item | <ul><li> |
| numbered_list_item | <ol><li> |
| to_do | <ul class="todo"><li><input type="checkbox"> |
| toggle | <details><summary> |
| code | <pre><code class="language-{lang}"> (Prism.js) |
| image | <figure><img><figcaption> |
| video | <div class="video-embed"><iframe> |
| embed | <iframe> (허용 목록 기반 필터링) |
| quote | <blockquote> |
| callout | <div class="callout"> |
| divider | <hr> |
| table | <table> |
| bookmark | <a class="bookmark-card"> (OG 데이터 fetch) |
| equation | KaTeX 렌더링 (빌드 타임) |
| column_list | <div class="columns"> (CSS Grid) |

Rich Text 인라인 변환:
| 노션 스타일 | HTML |
|-------------|------|
| bold | <strong> |
| italic | <em> |
| strikethrough | <del> |
| underline | <u> |
| code | <code> |
| color | <span style="color:..."> |
| link | <a href="..." rel="noopener"> |
```

#### Step 3: 이미지 처리 (CRITICAL)

```
⚠️ Notion 이미지 URL은 1시간 후 만료. 반드시 다운로드 필수.

1. 모든 이미지 URL 수집 (블록 이미지 + 커버 이미지)
2. 병렬 다운로드 (최대 5개 동시)
3. 이미지 처리:
   a. WebP 변환 (sharp 라이브러리)
   b. 리사이징: 최대 너비 1200px (원본이 더 작으면 유지)
   c. 썸네일 생성: 400px (글 목록 카드용)
   d. EXIF 데이터 제거 (개인정보 보호)
4. Cloudflare R2에 업로드:
   경로: {site_id}/images/{hash}.webp
   - 파일명을 콘텐츠 해시로 → 중복 업로드 방지
   - Cache-Control: public, max-age=31536000 (1년)
5. HTML 내 이미지 URL을 R2 URL로 교체
6. lazy loading 속성 추가: loading="lazy"
7. width/height 속성 추가 → CLS(Cumulative Layout Shift) 방지
```

#### Step 4: SEO 자동화

```
A. Slug 자동 생성 (노션 Slug 속성이 비어있을 때):
   1. 한글 → 영문 변환 (transliteration)
      라이브러리: slugify + transliteration
      예: "Next.js 시작하기" → "nextjs-sijaghagi"
   2. 특수문자 제거, 공백 → 하이픈
   3. 소문자 변환
   4. 최대 60자
   5. 중복 체크 → 중복 시 뒤에 -2, -3 등 추가

B. Meta Tags (각 포스트 HTML <head>에 삽입):
   <title>{meta_title || title} | {blog_name}</title>
   <meta name="description" content="{meta_description || 본문 첫 160자}">
   <meta name="robots" content="index, follow">
   <link rel="canonical" href="https://{domain}/{slug}">
   
   <!-- Open Graph -->
   <meta property="og:type" content="article">
   <meta property="og:title" content="{title}">
   <meta property="og:description" content="{description}">
   <meta property="og:image" content="{og_image_url}">
   <meta property="og:url" content="{canonical_url}">
   <meta property="og:site_name" content="{blog_name}">
   <meta property="article:published_time" content="{created_time}">
   <meta property="article:modified_time" content="{last_edited_time}">
   
   <!-- Twitter -->
   <meta name="twitter:card" content="summary_large_image">
   <meta name="twitter:title" content="{title}">
   <meta name="twitter:description" content="{description}">
   <meta name="twitter:image" content="{og_image_url}">

C. OG 이미지 자동 생성 (커버 이미지가 없는 포스트):
   - @vercel/og 또는 satori로 SVG → PNG 변환
   - 디자인: 블로그 이름 + 포스트 제목 + 그라데이션 배경
   - 크기: 1200×630px (OG 표준)
   - R2에 저장

D. sitemap.xml 생성:
   <?xml version="1.0" encoding="UTF-8"?>
   <urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
     <url>
       <loc>https://{domain}/</loc>
       <lastmod>{latest_post_date}</lastmod>
       <changefreq>daily</changefreq>
       <priority>1.0</priority>
     </url>
     <!-- 각 Published 포스트 -->
     <url>
       <loc>https://{domain}/{slug}</loc>
       <lastmod>{last_edited_time}</lastmod>
       <changefreq>weekly</changefreq>
       <priority>0.8</priority>
     </url>
   </urlset>

E. RSS Feed (rss.xml) 생성:
   - RSS 2.0 포맷
   - 최신 20개 포스트 포함
   - 각 아이템: title, link, description(첫 300자), pubDate, guid

F. robots.txt 생성:
   User-agent: *
   Allow: /
   Sitemap: https://{domain}/sitemap.xml

G. HTML 시맨틱 구조:
   <html lang="ko">
   <head>...</head>
   <body>
     <header>
       <nav>{블로그 이름, 메뉴}</nav>
     </header>
     <main>
       <article>
         <header>
           <h1>{title}</h1>
           <time datetime="{iso_date}">{formatted_date}</time>
           <span class="reading-time">{N}분</span>
         </header>
         <div class="content">
           {rendered_html}
         </div>
       </article>
       <aside class="toc">
         <nav>{목차}</nav>
       </aside>
     </main>
     <footer>{copyright, powered by Noble}</footer>
   </body>
   </html>
```

#### Step 5: TOC(목차) 생성

```
1. 렌더링된 HTML에서 h1/h2/h3 태그 추출
2. 각 헤딩에 id 부여: slugify(heading_text)
3. 중첩 구조 생성:
   H1
   ├── H2
   │   ├── H3
   │   └── H3
   └── H2
4. 3개 미만 헤딩이면 TOC 미생성
5. HTML 생성:
   <nav class="toc" aria-label="목차">
     <ol>
       <li><a href="#heading-1">제목 1</a>
         <ol>
           <li><a href="#heading-1-1">소제목 1-1</a></li>
         </ol>
       </li>
     </ol>
   </nav>
6. CSS:
   - 데스크톱: position: sticky, 우측 사이드바
   - 모바일: 글 상단 접이식 (details/summary)
   - 스크롤 하이라이트: IntersectionObserver (JS 최소)
```

#### Step 6: 정적 파일 생성

```
최종 출력 디렉토리 구조:
dist/
├── index.html              ← 글 목록 (홈)
├── {slug}/index.html       ← 각 포스트
├── sitemap.xml
├── rss.xml
├── robots.txt
├── css/
│   ├── style.css           ← 메인 스타일
│   └── prism.css           ← 코드 하이라이팅
├── js/
│   └── toc.js              ← TOC 스크롤 하이라이트 (< 2KB)
└── images/                  ← OG 이미지 등 (R2 외부 저장 이미지 제외)
```

#### Step 7: Cloudflare Pages 배포

```
배포 방식: Cloudflare Pages Direct Upload API

1. dist/ 디렉토리를 ZIP으로 압축
2. Cloudflare API 호출:
   POST https://api.cloudflare.com/client/v4/accounts/{account_id}/pages/projects/{project_name}/deployments
   Headers: Authorization: Bearer {CF_API_TOKEN}
   Body: FormData with ZIP file
3. 배포 완료 대기 (폴링)
4. 커스텀 도메인이 설정되어 있으면 자동 라우팅

배포 시간 목표: 30초 이내 (50 포스트 기준)
```

### 5.3 빌드 에러 처리

| 에러 | 감지 | 사용자 메시지 | 복구 |
|------|------|-------------|------|
| Notion 401 | API 응답 | "노션 재연동이 필요합니다" | 재연동 버튼 |
| Notion 429 (Rate Limit) | API 응답 | "잠시 후 다시 시도해주세요" | 자동 재시도 (3회, 10초 간격) |
| Notion API Down | timeout 30s | "노션 서버에 연결할 수 없습니다" | 1분 후 재시도 |
| 이미지 다운로드 실패 | HTTP 에러/timeout | 실패 이미지 스킵, 빌드 계속 | placeholder 이미지 삽입 |
| Cloudflare 배포 실패 | API 에러 | "배포에 실패했습니다" | 자동 재시도 (2회) |
| 빌드 타임아웃 | 5분 초과 | "빌드 시간이 초과되었습니다" | 포스트 수 줄이기 안내 |

에러 발생 시에도 마지막 성공 빌드가 유지되어 블로그는 정상 서빙.

---

## 6. 프론트엔드 (대시보드) 상세

### 6.1 화면 구조

```
Noble Dashboard
├── /login, /signup        ─── 인증
├── /onboarding            ─── 4단계 위저드 (최초 1회)
├── /dashboard             ─── 메인 (블로그 요약 + 배포)
├── /posts                 ─── 포스트 목록 + SEO 상태
└── /settings
    ├── 일반               ─── 블로그 이름, URL, 도메인, 노션 연동
    ├── SEO                ─── sitemap 복사, 서치콘솔 가이드
    ├── 테마               ─── (Phase 1 - Coming Soon)
    └── 결제               ─── 플랜, Paddle 포털, 환불
```

### 6.2 인증 화면 (`/login`, `/signup`)

| 항목 | 스펙 |
|------|------|
| **방식** | Supabase Auth: 이메일(Magic Link) + Google OAuth |
| **비밀번호** | 없음. Magic Link 전용 (보안 + 개발 공수 절감) |
| **회원가입 시** | `trial_ends_at = NOW() + 14 days` 자동 설정 |
| **로그인 후** | `onboarding_completed === false` → `/onboarding` / `true` → `/dashboard` |
| **태그라인** | "노션으로 글만 쓰면, 구글에 잘 잡히는 블로그" |
| **신뢰 문구** | "14일 무료 체험 · 카드 등록 불필요" |

### 6.3 온보딩 위저드 (`/onboarding`)

4단계 스텝 위저드. 각 단계 완료 시 ✅ 표시, 프로그레스 바 시각 표시.

#### Step 1: 노션 연동

| 항목 | 스펙 |
|------|------|
| **메인 CTA** | "노션 연동하기" 버튼 → Notion OAuth 화면 이동 |
| **안내 문구** | OAuth 버튼 위: "Notion 공식 API를 사용하여 안전하게 연동됩니다" |
| **사전 가이드** | "블로그로 사용할 데이터베이스만 선택해 주세요" |
| **URL 폴백** | "연동이 안 되시나요?" 클릭 → 숨겨진 URL 입력 필드 토글 |
| **URL 파싱** | 정규식: `/([a-f0-9]{32})/` (32자 hex, `?v=` 이전까지) |
| **Page vs DB** | Notion API `object` 필드 체크. page이면 → "데이터베이스 전체 주소를 입력해 주세요" |
| **완료 조건** | access_token 저장 성공 |

OAuth URL 구성:
```
https://api.notion.com/v1/oauth/authorize
  ?client_id={NOTION_CLIENT_ID}
  &response_type=code
  &owner=user
  &redirect_uri={BASE_URL}/api/auth/callback/notion
  &state={csrf_token}
```

#### Step 2: DB 선택

| 항목 | 스펙 |
|------|------|
| **데이터** | Notion API로 권한 부여된 DB 목록 조회 (`/v1/search`, filter: database) |
| **표시 정보** | DB 이름 + 페이지 수 |
| **자동 선택** | DB가 1개면 자동 선택 + 다음 단계 이동 |
| **선택 제한** | 1개만 (Phase 0) |
| **템플릿** | "처음부터 시작하고 싶으신가요? 노블 블로그 템플릿 복제하기" |
| **중복 방지** | 이미 다른 계정에서 연동된 DB면 → "이미 연동된 데이터베이스입니다" 에러 |

#### Step 3: 속성 확인

| 항목 | 스펙 |
|------|------|
| **검사 속성** | Title(제목), Slug(텍스트), Status(셀렉트) |
| **검사 방법** | `GET /v1/databases/{db_id}` → properties 필드에서 확인 |
| **Title** | 모든 DB에 기본 존재 → 항상 ✅ |
| **Slug 검사** | properties 중 type: "rich_text"이고 이름이 "Slug"인 속성 |
| **Status 검사** | properties 중 type: "select"이고 이름이 "Status"인 속성 |
| **Status 옵션** | "Published" 옵션 존재 여부 추가 확인 |
| **없을 때** | 구체적 안내: "노션에서 '블로그 포스트' DB를 열고 텍스트 타입의 'Slug' 속성을 추가해주세요" |
| **재확인** | "다시 확인하기" 버튼 → 속성 재검사 |
| **자동 진행** | 모든 ✅이면 2초 뒤 자동으로 Step 4 이동 |

#### Step 4: 블로그 URL 설정

| 항목 | 스펙 |
|------|------|
| **서브도메인** | `{입력값}.noble.blog` |
| **서브도메인 규칙** | 3~30자, 영문 소문자+숫자+하이픈, 하이픈으로 시작/끝 불가 |
| **중복 체크** | 실시간 (debounce 500ms) → "사용 가능" / "이미 사용 중" |
| **커스텀 도메인** | 선택사항. 입력 시 DNS 설정 안내 표시 |
| **DNS 안내** | "CNAME 레코드를 추가해주세요: noble-cdn.pages.dev" |
| **DNS 검증** | "DNS 확인하기" 버튼 → DNS lookup 실행 → 결과 표시 |
| **완료 CTA** | "블로그 만들기" → 사이트 생성 + 첫 빌드 트리거 + `/dashboard` 이동 |

### 6.4 메인 대시보드 (`/dashboard`)

사용자가 매일 보는 핵심 화면. "현재 상태 + 배포" 2가지만 집중.

#### 블로그 요약 카드

| 항목 | 데이터 소스 | 표시 형식 |
|------|-----------|----------|
| 블로그 URL | sites.subdomain / custom_domain | 클릭 가능 외부 링크 |
| 발행된 글 | posts WHERE status='Published' COUNT | "12개" |
| 초안 | posts WHERE status='Draft' COUNT | "3개" |
| 마지막 배포 | sites.last_build_at | 상대 시간 ("2시간 전") |
| 블로그 상태 | sites.last_build_status | 🟢 정상 / 🟡 빌드중 / 🔴 실패 |

#### SEO 현황 카드

| 항목 | 동작 |
|------|------|
| Sitemap URL | `https://{domain}/sitemap.xml` + 📋 복사 버튼 |
| RSS URL | `https://{domain}/rss.xml` + 📋 복사 버튼 |
| 서치콘솔 가이드 | 외부 링크 (Google/Naver 각각) |

#### 체험/결제 카드

| 상태 | 표시 내용 |
|------|----------|
| **체험 중** | "남은 기간: N일" + 프로그레스 바 + "업그레이드" CTA |
| **체험 D-3** | 배너: "체험 기간이 3일 남았습니다" (알림 배너에도 표시) |
| **체험 만료** | "체험이 종료되었습니다. 업그레이드하여 블로그를 계속하세요" |
| **유료 (정상)** | "Pro 플랜 · 다음 결제일: YYYY.MM.DD" |
| **유료 (past_due)** | "결제에 실패했습니다. 결제 수단을 확인해주세요." + Paddle 링크 |

#### 알림 배너 (조건부, 우선순위 순)

| 우선순위 | 조건 | 메시지 | CTA |
|---------|------|-------|-----|
| 1 🔴 | last_build_status = 'failed' | "마지막 배포가 실패했습니다" | [상세 보기] |
| 2 🟡 | Notion API 401 감지 | "노션 연동이 해제되었습니다" | [재연동하기] |
| 3 🟡 | trial D-3 이하 | "체험 기간이 N일 남았습니다" | [업그레이드] |
| 4 🟡 | past_due | "결제에 실패했습니다" | [결제 수단 확인] |
| 5 🔵 | 새 기능 출시 | "새 기능이 추가되었습니다!" | [알아보기] |

동시 발생 시 최고 우선순위 1개만 표시.

### 6.5 포스트 관리 (`/posts`)

| 항목 | 스펙 |
|------|------|
| **데이터** | 마지막 빌드 시 캐시된 posts 테이블 데이터 (실시간 아님) |
| **필터 탭** | 전체 / Published / Draft — 탭 클릭 필터 |
| **테이블 컬럼** | 제목, Slug, Status, 수정일(상대시간), SEO 상태 |
| **SEO 상태** | 각 글 아래: ✅/⚠️/❌ 아이콘 (meta, slug, image 체크) |
| **글 클릭** | 해당 노션 페이지 새 탭 열기 (notion_url) |
| **빈 Slug** | "(없음)" + 주황색 경고 표시, "배포 시 자동 생성됩니다" 툴팁 |
| **정렬** | 수정일 최신순 고정 |
| **페이지네이션** | 50개 초과 시 "더 보기" 버튼 |
| **안내 문구** | 하단: "글 편집은 노션에서 직접 하세요. 편집 후 배포 버튼을 누르면 반영됩니다." |

### 6.6 설정 (`/settings`)

4개 탭: 일반, SEO, 테마(Phase 1), 결제

#### 일반 탭

| 필드 | 타입 | 제한/규칙 |
|------|------|----------|
| 블로그 이름 | text input | 1~50자 |
| 블로그 설명 | textarea | 0~300자, meta description 기본값 |
| 서브도메인 | text (읽기 전용) + [변경] 버튼 | 변경 시 중복 체크 |
| 커스텀 도메인 | text input | 소문자 변환, DNS 검증 버튼 |
| DNS 상태 | 읽기 전용 | ✅ 확인됨 / ❌ 미확인 / ⏳ 확인 중 |
| 노션 연동 | 읽기 전용 | 워크스페이스 이름, DB 이름, [연동 해제] [다른 DB 선택] |

#### SEO 탭

| 필드 | 동작 |
|------|------|
| Sitemap URL | 읽기 전용 + 📋 복사 |
| RSS URL | 읽기 전용 + 📋 복사 |
| 서치콘솔 가이드 | 외부 링크 (Google, Naver) |
| OG 이미지 | 라디오: 노션 커버(기본) / 커스텀 업로드 |
| 기본 Meta Description | textarea, 0~160자 |
| robots.txt | 체크박스: 크롤링 허용(기본 ON) / 차단 |

#### 테마 탭 (Phase 1)

Phase 0: "Coming Soon" 배지 + 3개 테마 프리뷰 이미지 (미니멀, 매거진, 다크). 클릭 불가.

#### 결제 탭

| 상태 | 표시 내용 |
|------|----------|
| **체험 중** | 남은 일수 + Pro/Business 플랜 카드 2개 + [시작하기] CTA |
| **유료** | 현재 플랜 이름, 금액, 다음 결제일, [업그레이드] [연간 전환] |
| **Paddle 관리** | [Paddle 결제 관리 열기] → paddle.net 외부 링크 |
| **구독 취소** | [구독 취소하기] → 확인 모달 → Paddle cancel API |
| **환불** | [환불 요청하기] → 14일 이내 자동승인, 초과 시 "검토 후 안내" |

### 6.7 글로벌 컴포넌트

#### 사이드바

| 메뉴 | 경로 | 아이콘 |
|------|------|-------|
| 대시보드 | /dashboard | 📊 |
| 포스트 | /posts | 📝 |
| 설정 | /settings | ⚙️ |
| --- (구분선) | | |
| 내 블로그 | 외부 링크 (새 탭) | 🔗 |
| 도움말 | 외부 링크 | 📖 |
| --- (구분선) | | |
| 플랜 정보 | 텍스트 표시 | |
| 이메일 + 로그아웃 | | |

반응형:
- 1024px+: 고정 240px
- 768~1023px: 60px (아이콘만)
- ~767px: 숨김, 햄버거 메뉴 → 오버레이

#### 배포 FAB (Floating Action Button)

| 항목 | 스펙 |
|------|------|
| **위치** | 전체 화면 우하단 고정 (bottom: 24px, right: 24px) |
| **노출** | 온보딩 제외 모든 화면에 표시 |
| **데스크톱** | 48px 높이 + "🚀 배포" 텍스트 |
| **모바일** | 56×56px 원형, 🚀 아이콘만 |
| **호버 툴팁** | "오늘 남은 배포: 8/10" |
| **상태: 기본** | 파란색, 클릭 가능 |
| **상태: 빌드중** | 회색, 비활성 + 스피너 + "배포 중..." |
| **상태: 완료** | 초록색, "배포 완료!" → 3초 후 기본 복귀 |
| **상태: 실패** | 빨간색, "배포 실패" → 클릭 시 에러 상세 |
| **상태: 제한** | 회색, "오늘 배포 N/N" |
| **z-index** | 최상위 (모달 아래) |

#### 토스트 알림

| 이벤트 | 타입 | 메시지 | 지속 시간 |
|--------|------|-------|----------|
| 배포 완료 | success | "배포 완료! 블로그가 업데이트되었습니다." | 5초 |
| 배포 실패 | error | "배포에 실패했습니다. {error_message}" | 10초 (수동 닫기) |
| 설정 저장 | success | "설정이 저장되었습니다." | 3초 |
| 클립보드 복사 | info | "복사되었습니다." | 2초 |
| 노션 연동 | success | "노션이 연결되었습니다!" | 3초 |

---

## 7. Notion 연동 상세

### 7.1 Notion Integration 설정

```
타입: Public Integration
이름: Noble Blog
설명: "노션 데이터베이스를 SEO 최적화된 블로그로 변환합니다"
리다이렉트 URL: https://{domain}/api/auth/callback/notion
권한 (Phase 0): Read content
권한 (Phase 1+): Read content, Update content (Auto-Fix용)
```

### 7.2 필수 Notion DB 구조

```
| 속성 이름 | 타입 | 필수 | 용도 |
|-----------|------|------|------|
| Title     | title | ✅ (자동 존재) | 포스트 제목, H1, meta title |
| Slug      | rich_text | ✅ | URL 경로 (/my-post) |
| Status    | select | ✅ | "Published"만 블로그에 표시 |
| Category  | select | Phase 1 | 카테고리 페이지 생성 |
| Tags      | multi_select | Phase 1 | 태그 페이지 생성 |
| Meta Description | rich_text | 선택 | 커스텀 SEO description |
| Meta Title | rich_text | 선택 | 커스텀 SEO title |
```

### 7.3 Noble 공식 Notion 템플릿

```
제공 방식: Notion 템플릿 갤러리 링크 (또는 공개 페이지 복제 링크)
포함 내용:
  - Title, Slug, Status, Category, Tags, Meta Description 속성 사전 구성
  - Status 옵션: Published, Draft, Idea
  - 예시 포스트 3개 포함 (온보딩 후 즉시 빌드 테스트 가능)
  - _noble_version 속성 (숨김, 값: "1.0") — Phase 2 버전 관리용
```

### 7.4 Notion API 호출 패턴

```
Rate Limit 대응:
  - 기본 간격: 350ms/요청 (Notion 제한: 3 req/s)
  - 429 응답 시: Retry-After 헤더 값만큼 대기 후 재시도
  - 최대 재시도: 3회
  - 전체 빌드 타임아웃: 5분

에러 처리:
  - 401 → 토큰 만료: 사이트에 플래그 세팅, 대시보드 배너 표시
  - 403 → 권한 부족: "노션에서 해당 페이지 접근 권한을 확인해주세요"
  - 404 → DB 삭제됨: "연결된 데이터베이스를 찾을 수 없습니다"
  - 500/502/503 → Notion 서버 장애: 자동 재시도 → 실패 시 "노션 서버 장애"
```

---

## 8. 결제 시스템 (Paddle)

### 8.1 Paddle 설정

```
모드: Paddle Billing (신규 API)
환경: Sandbox (개발) → Production (런칭)

Products:
  - Noble Pro Monthly
  - Noble Pro Annual
  - Noble Business Monthly
  - Noble Business Annual

각 Product에 Price 설정:
  - Pro Monthly: $9/월 (recurring)
  - Pro Annual: $86/년 (recurring)
  - Business Monthly: $19/월 (recurring)
  - Business Annual: $182/년 (recurring)

Checkout 설정:
  - Paddle.js overlay checkout (인라인 아님)
  - success_url: /settings?tab=billing&checkout=success
  - 사용자 식별: customer_email + custom_data.user_id
```

### 8.2 플랜 및 가격

| 플랜 | 월간 | 연간 (20% 할인) | Paddle 수수료 | 실수령 |
|------|------|---------------|-------------|--------|
| **Pro** | $9/월 | $86/년 | $0.95/월 | $8.05/월 |
| **Business** | $19/월 | $182/년 | $1.45/월 | $17.55/월 |

### 8.3 체험 기간 로직

```
체험 시작: 회원가입 시 자동 (trial_ends_at = NOW() + 14 days)
체험 중 기능: Pro 전체 기능 (일 3회 배포 제한)
체험 만료 체크:
  - 매 API 요청 시: trial_ends_at < NOW() 이면 → 기능 차단
  - 빌드 요청 시: 402 TRIAL_EXPIRED 반환
체험 만료 후:
  - 블로그 서빙 중지 (Cloudflare 배포 유지하되 커스텀 도메인 해제)
  - 대시보드 접근 가능 (업그레이드 유도)
  - 데이터 30일 보관 → 이후 삭제
알림:
  - D-3: 대시보드 배너 + 이메일
  - D-1: 이메일
  - D+0: 이메일 (만료 안내 + 업그레이드 링크)
```

### 8.4 배포 횟수 제한 로직

```typescript
async function checkDeployLimit(site: Site, plan: string): Promise<{
  allowed: boolean;
  used: number;
  limit: number;
}> {
  const today = new Date().toISOString().split('T')[0];
  
  // 날짜 변경 시 리셋
  if (site.daily_deploy_date !== today) {
    await updateSite(site.id, { daily_deploy_count: 0, daily_deploy_date: today });
    site.daily_deploy_count = 0;
  }
  
  const limits: Record<string, number> = {
    trial: 3,
    pro: 10,
    business: Infinity,
  };
  
  const limit = limits[plan] ?? 3;
  
  return {
    allowed: site.daily_deploy_count < limit,
    used: site.daily_deploy_count,
    limit,
  };
}
```

### 8.5 환불 정책 (상세)

#### 전액 환불

| 조건 | 처리 |
|------|------|
| 최초 구매 14일 이내 | 자동 승인. `Paddle Adjustment API → action: refund, type: full` |
| Noble 장애 48시간+ | 수동 검토 후 승인. 해당 월 전액 환불 |

#### 부분 환불

연간 결제 중도 해지 계산:
```
사용 기간: N개월
정가 기준 사용 금액: $9 × N (연간 할인 소급 취소)
환불 금액: 연간 결제액 - (정가 × 사용 개월)

예: Pro 연간($86) 5개월 사용
→ $86 - ($9 × 5) = $86 - $45 = $41 환불
```

#### 환불 프로세스

```
1. 사용자: 대시보드 [환불 요청] 클릭
2. 클라이언트:
   - 클릭 즉시 버튼 disabled (더블클릭 방지)
   - Idempotency-Key (UUID v4) 생성 → 헤더에 포함
3. 서버 (POST /api/paddle/refund):
   a. Idempotency-Key로 idempotency_keys 테이블 조회
      → 존재하면: 캐시된 응답 반환 (이중 환불 방지)
   b. 현재 구독 상태 확인:
      → 이미 환불 처리됨 → "이미 환불이 진행 중입니다" 반환
   c. 구매 14일 이내? → Paddle Adjustment API 자동 호출
   d. 14일 초과? → 관리자 알림 (이메일) → 수동 검토
   e. 응답을 idempotency_keys에 저장
4. Paddle: 원래 결제 수단 환불 처리 (카드 3~5일, PayPal 48시간)
5. Paddle: Credit Note PDF 자동 발송
6. 시스템: 구독 즉시 해지 + 30일 데이터 보관
```

### 8.6 구독 취소

| 항목 | 스펙 |
|------|------|
| 취소 시점 | 언제든 가능 |
| 취소 효력 | 현재 결제 기간 종료까지 서비스 이용 가능 |
| 자동 갱신 방지 | 갱신일 48시간 전까지 취소 필요 (Paddle 정책) |
| 데이터 보관 | 종료 후 30일 → 이후 영구 삭제 |
| 재구독 | 30일 이내: 기존 데이터 즉시 복구 |
| 구현 | `Paddle API → POST /subscriptions/{id}/cancel → effective_from: "next_billing_period"` |
| **멱등성** | 서버에서 `paddle_subscription_status` 사전 체크. 이미 `canceled`면 → "이미 취소되었습니다" 반환 (Paddle API 재호출 안 함). 클라이언트 버튼도 클릭 즉시 disabled |

---

## 9. 커스텀 도메인

### 9.1 설정 플로우

```
1. 사용자: 설정 > 일반 > 커스텀 도메인에 도메인 입력
   예: blog.mydomain.com

2. 시스템: CNAME 레코드 안내 표시
   "DNS 설정에서 CNAME 레코드를 추가해주세요"
   Type: CNAME
   Name: blog (또는 @)
   Target: noble-cdn.pages.dev

3. 사용자: DNS 설정 후 [DNS 확인하기] 클릭

4. 시스템:
   a. DNS lookup으로 CNAME 확인
   b. Cloudflare Pages API로 커스텀 도메인 등록
   c. SSL 인증서 자동 발급 (Cloudflare Universal SSL)
   d. 결과: ✅ DNS 확인됨 · SSL 활성 또는 ❌ CNAME을 찾을 수 없습니다

5. DNS 전파 지연 안내:
   "DNS 변경이 반영되기까지 최대 48시간이 걸릴 수 있습니다.
    보통 몇 분에서 몇 시간 이내에 완료됩니다."
```

### 9.2 서브도메인 규칙

```
형식: {subdomain}.noble.blog
규칙:
  - 3~30자
  - 영문 소문자 + 숫자 + 하이픈(-)
  - 하이픈으로 시작/끝 불가
  - 예약어 차단: admin, api, www, app, dashboard, blog, help, support, status
```

---

## 10. 보안

### 10.1 인증/인가

| 항목 | 구현 |
|------|------|
| 인증 | Supabase Auth JWT (access_token + refresh_token) |
| 인가 | Supabase RLS (Row Level Security) — 자기 데이터만 접근 |
| API 보호 | 모든 API Route에서 `getUser()` 호출 → 미인증 시 401 |
| CSRF | Notion OAuth state 파라미터 + SameSite cookie |
| XSS | Next.js 기본 escape + CSP 헤더 |

### 10.2 데이터 보호

| 항목 | 구현 |
|------|------|
| Notion 토큰 | AES-256-GCM 암호화 저장. 암호화 키는 환경변수 |
| DB 접근 | Supabase RLS 활성. 서비스 역할 키는 서버 사이드만 |
| HTTPS | Vercel(대시보드) + Cloudflare(블로그) 모두 자동 SSL |
| 이미지 | 사용자 업로드 이미지 → 바이러스 스캔 없음 (Phase 0), R2에 격리 저장 |

### 10.3 Paddle 웹훅 보안

```
1. Paddle-Signature 헤더에서 ts와 h1 추출
2. ts가 현재 시간 ±5분 이내인지 확인 (replay 방지)
3. HMAC-SHA256(webhook_secret, ts + ":" + raw_body) === h1 검증
4. 검증 실패 시 403 반환
5. event_id 중복 확인 (멱등성)
```

---

## 10.4 멱등성 정책 (Idempotency)

> **원칙: "100번 요청해도 결과는 1번과 같다"**
> 중복 실행되면 큰일 나는 모든 POST 엔드포인트에 멱등성을 보장한다.

### 방어 계층 (3중 방어)

```
Layer 1 — 클라이언트 (즉시 방어):
  - 버튼 클릭 즉시 disabled (더블클릭 방지)
  - loading 상태 표시
  - 응답 전까지 재클릭 차단

Layer 2 — Idempotency Key (서버 방어):
  - 클라이언트가 UUID v4를 생성 → Idempotency-Key 헤더로 전송
  - 서버가 idempotency_keys 테이블에서 키 조회
  - 존재하면 → 캐시된 응답 반환 (실제 로직 실행 안 함)
  - 없으면 → 로직 실행 + 응답 저장
  - 키 유효기간: 24시간

Layer 3 — DB 제약 (최종 방어):
  - sites: one_site_per_user UNIQUE (user_id)
  - sites: notion_database_id UNIQUE
  - processed_webhooks: event_id PRIMARY KEY
  - builds: 진행 중 빌드 존재 여부 체크 (status IN ('queued','building','deploying'))
```

### 엔드포인트별 적용

| 엔드포인트 | Layer 1 | Layer 2 | Layer 3 | 비고 |
|-----------|---------|---------|---------|------|
| POST `/api/build/trigger` | ✅ FAB disabled | ✅ Idempotency-Key | ✅ 진행 중 빌드 체크 | 🔴 핵심 |
| POST `/api/site` | ✅ 버튼 disabled | ✅ user_id 체크 | ✅ UNIQUE 제약 | 🔴 핵심 |
| POST `/api/paddle/refund` | ✅ 버튼 disabled | ✅ Idempotency-Key | ✅ 환불 상태 체크 | 🔴 돈 관련 |
| POST `/api/paddle/cancel` | ✅ 버튼 disabled | - | ✅ status 사전 체크 | 🟡 이미 canceled면 스킵 |
| POST `/api/paddle/webhook` | - (서버 간) | - | ✅ event_id 중복 체크 | 🔴 핵심 |
| GET `/api/auth/callback/notion` | - (리다이렉트) | - | ✅ 기존 토큰 체크 + code 일회용 | 🟡 뒤로가기 방어 |
| PATCH `/api/site` | - | - | - | 🟢 본질적 멱등 (PUT/PATCH) |
| GET 전체 | - | - | - | 🟢 본질적 멱등 (조회) |

### Idempotency Key 구현 패턴

```typescript
// lib/idempotency.ts
export async function withIdempotency<T>(
  key: string,
  userId: string,
  endpoint: string,
  fn: () => Promise<{ status: number; body: T }>
): Promise<{ status: number; body: T }> {
  // 1. 기존 키 조회
  const existing = await supabase
    .from('idempotency_keys')
    .select('response_status, response_body')
    .eq('key', key)
    .single();

  if (existing.data) {
    // 캐시된 응답 반환
    return {
      status: existing.data.response_status,
      body: existing.data.response_body as T
    };
  }

  // 2. 실제 로직 실행
  const result = await fn();

  // 3. 응답 저장
  await supabase.from('idempotency_keys').insert({
    key,
    user_id: userId,
    endpoint,
    response_status: result.status,
    response_body: result.body
  });

  return result;
}
```

---

## 11. 에러 처리 & 복원력

### 11.1 에러 분류

| 카테고리 | 예시 | 사용자 영향 | 대응 |
|---------|------|-----------|------|
| **차단적** | Notion 401, 체험 만료 | 빌드 불가 | 배너 + 명확한 해결 안내 |
| **일시적** | Notion 429/500, CF 배포 실패 | 잠시 빌드 불가 | 자동 재시도 + "잠시 후 다시" 안내 |
| **부분적** | 일부 이미지 다운로드 실패 | 일부 이미지 깨짐 | 스킵 + placeholder + 경고 |
| **정보성** | Slug 미설정, 이미지 alt 없음 | SEO 점수 감소 | 포스트 목록에 ⚠️ 표시 |

### 11.2 핵심 복원력 원칙

```
1. 마지막 성공 빌드 보존:
   빌드 실패 시에도 이전 배포가 유지됨.
   → 사용자 블로그는 항상 접속 가능.

2. 노션 장애 대응:
   빌드 시에만 Notion API 호출.
   → Notion 장애 시 기존 블로그 정상 서빙.

3. Graceful Degradation:
   이미지 실패 → placeholder
   OG 이미지 생성 실패 → 기본 이미지
   코드 하이라이팅 실패 → plain code block
```

---

## 12. 리스크 체크리스트

| 리스크 | 확률 | 영향 | 대응 |
|--------|------|------|------|
| Notion API Rate Limit | 낮음 | 빌드 지연 | 빌드 타임 전용 (실시간 아님), 350ms 간격 |
| Notion API 장애 | 중간 | 빌드 불가 | 캐시 유지, 장애 시 블로그 정상 서빙 |
| Notion OAuth 토큰 revoke | 중간 | 빌드 불가 | 401 감지 → 재연동 배너 |
| "검토받지 않은 앱" 경고 | 높음 | 사용자 불안 | 안내 문구 + 유료 전환 시 Notion 리뷰 신청 |
| Notion API Terms 변경 | 낮음 | 사업 중단 | 상업 사용 허용 확인, TOS 모니터링 |
| Cloudflare 무료 한도 | 중간 | 빌드 한도 | 500빌드/월, 50유저 후 Pro($20) 검토 |
| Notion 이미지 URL 만료 | 확실 | 이미지 깨짐 | 빌드 시 다운로드 → R2 저장 (필수) |
| Paddle 계정 승인 지연 | 중간 | 결제 불가 | 사전 신청, 체험 기간으로 버퍼 |
| 사용자 Notion 구조 다양성 | 높음 | 빌드 에러 | 엄격한 속성 검증 + 명확한 에러 메시지 |

---

## 13. 개발 로드맵

### Phase 0 — MVP (목표: 5일)

```
Day 1: 프로젝트 셋업 + Notion OAuth + 토큰 암호화 저장 + DB 선택
Day 2: Notion API → 마크다운 변환 → 정적 HTML 빌드 파이프라인
Day 3: SEO 자동화 (slug, meta, sitemap, RSS, robots.txt, OG) + 이미지 처리
Day 4: 대시보드 UI (온보딩 + 메인 + 포스트 + 설정) + 배포 FAB + 커스텀 도메인
Day 5: Paddle 연동 + QA + 랜딩 페이지 + 런칭
```

Phase 0 완료 기준 (Definition of Done):
- [ ] 노션 OAuth 연동 + DB 선택 + 속성 검증 작동
- [ ] 온보딩 4단계 위저드 완료
- [ ] 배포 버튼 → 노션 → 정적 HTML 빌드 → Cloudflare 배포 완료
- [ ] sitemap, RSS, robots.txt 자동 생성
- [ ] 커스텀 도메인 + SSL 작동
- [ ] Lighthouse 성능 90+ 달성
- [ ] Paddle checkout → 구독 활성화 → 웹훅 처리
- [ ] 14일 체험 + 배포 횟수 제한 작동
- [ ] 대시보드 5개 화면 모두 작동

### Phase 1 — 유저 10명 확보 후

| 기능 | 설명 | 우선순위 |
|------|------|---------|
| 페이지뷰 통계 | Umami 연동 또는 자체 경량 analytics | P1 |
| AI Meta Description | Claude Haiku API, 글당 ~$0.01 | P1 |
| JSON-LD 구조화 데이터 | Article, BlogPosting, Breadcrumb | P1 |
| 이전/다음글 네비게이션 | 포스트 하단 자동 생성 | P1 |
| 카테고리/태그 | Notion Select/Multi-select 연동 | P1 |
| 테마 3종 | 미니멀, 매거진, 다크 + 컬러/폰트 커스텀 | P2 |
| 뉴스레터 구독 폼 | Buttondown 또는 자체 구현 | P2 |
| 댓글 | Giscus (GitHub Discussions 기반) | P2 |
| 온보딩 GIF 가이드 | 각 단계별 5초 GIF + Confetti | P2 |
| Auto-Fix | OAuth Read+Write, 속성 자동 생성 | P3 |

### Phase 2 — 유료 전환 후

| 기능 | 설명 | 플랜 |
|------|------|------|
| Google AdSense 자동 배치 | 코드 입력 → 헤더/중간/푸터 배치 | Business |
| 제휴 링크 관리 | 외부 링크 nofollow + 관리 페이지 | Business |
| 멤버십/유료 구독 | Paddle 연동 결제벽 | Business |
| 스폰서 배너 | 배너 업로드 → 자동 배치 | Business |
| 멀티 블로그 | 1계정 다수 블로그 | Business |
| SEO 점수 | 글별 체크리스트 점수 | Pro |
| 내부 링크 제안 | AI 기반 | Business |
| 서치콘솔 대시보드 | Google API 연동 | Business |
| 관련 글 추천 | 태그 기반 → AI 임베딩 | Pro |

---

## 14. 성공 지표 (KPI)

### Phase 0

| 지표 | 목표 | 측정 |
|------|------|------|
| 온보딩 완료율 | >70% | 회원가입 → 첫 빌드 완료 비율 |
| 온보딩 소요 시간 | <3분 | 첫 화면 → 첫 빌드 시작까지 |
| Lighthouse 성능 | 90+ | 자동 테스트 |
| 빌드 성공률 | >95% | builds 테이블 성공/실패 비율 |
| 빌드 소요 시간 | <30초 (50 포스트) | builds.build_duration_ms |

### Phase 1

| 지표 | 목표 | 측정 |
|------|------|------|
| WAU (Weekly Active Users) | 상승 추세 | 주간 빌드 실행 유저 수 |
| 체험→유료 전환율 | >5% | 14일 체험 완료 후 결제 비율 |
| 월간 이탈률 | <10% | 유료 구독 취소율 |

---

## 15. 환경 변수 목록

```env
# === Supabase ===
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# === Notion ===
NOTION_CLIENT_ID=
NOTION_CLIENT_SECRET=
NOTION_REDIRECT_URI=

# === 암호화 ===
ENCRYPTION_KEY=                    # 64자 hex (32바이트 AES-256 키)

# === Cloudflare ===
CLOUDFLARE_ACCOUNT_ID=
CLOUDFLARE_API_TOKEN=
CLOUDFLARE_R2_ACCESS_KEY_ID=
CLOUDFLARE_R2_SECRET_ACCESS_KEY=
CLOUDFLARE_R2_BUCKET=

# === Paddle ===
PADDLE_API_KEY=
PADDLE_WEBHOOK_SECRET=
PADDLE_CLIENT_TOKEN=               # 프론트엔드 Paddle.js용
PADDLE_PRO_MONTHLY_PRICE_ID=
PADDLE_PRO_ANNUAL_PRICE_ID=
PADDLE_BUSINESS_MONTHLY_PRICE_ID=
PADDLE_BUSINESS_ANNUAL_PRICE_ID=

# === 기타 ===
NEXT_PUBLIC_BASE_URL=              # https://app.noble.blog
SENTRY_DSN=                        # 에러 트래킹 (선택)
RESEND_API_KEY=                    # 이메일 발송 (선택)
```

---

*Noble PRD v2.0 — 2025-02-13*
*Ship fast, iterate based on user feedback.*
