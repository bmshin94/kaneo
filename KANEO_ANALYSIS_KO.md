# Kaneo 분석 정리 (한국어)

> 이 저장소가 무엇인지, 어떻게 설치·사용하는지, 수익화 가능성과 PHP 전환 검토까지
> 한 번에 정리한 문서입니다.

## 저장소 주소

| 구분 | 주소 |
| --- | --- |
| 이 저장소 (포크) | https://github.com/bmshin94/kaneo |
| 원본 (업스트림) | https://github.com/usekaneo/kaneo |
| 공식 웹사이트 | https://kaneo.app |
| 공식 문서 | https://kaneo.app/docs/core |
| 클라우드 (본가 운영) | https://cloud.kaneo.app |
| 배포 CLI (drim) | https://github.com/usekaneo/drim |
| MCP 패키지 (npm) | https://www.npmjs.com/package/@kaneo/mcp |
| 커뮤니티 (Discord) | https://discord.gg/rU4tSyhXXU |

---

## 1. 이게 무엇인가

**Kaneo** (/kəˈneɪ.oʊ/) 는 **오픈소스 셀프호스팅 프로젝트 관리 툴**입니다.
쉽게 말해 **"내 서버에 직접 설치해서 쓰는 Jira / Trello / Linear"** 입니다.

- 라이선스: **MIT** (Copyright (c) 2024 Andrej Acevski) — 상업적 사용·수정·판매 가능
- 철학: *"툴이 기능이 부족한 게 문제가 아니라 너무 많은 게 문제다. 좋은 툴은 보이지 않아야 한다"*
- 핵심 차별점: **셀프호스팅** = 데이터가 외부 SaaS로 나가지 않음

### 일반 SaaS와의 비교

| | Trello / Jira | **Kaneo** |
| --- | --- | --- |
| 데이터 저장 위치 | 벤더 서버 (해외) | **내 서버** |
| 비용 | 인원 늘면 유료 | **무료** (셀프호스팅) |
| 소스코드 | 비공개 | **전체 공개 (MIT)** |

---

## 2. 폴더 구조

pnpm + Turborepo 기반 **모노레포** 구조입니다.

```
apps/
 ├─ api/    백엔드 API      (Hono + Drizzle ORM + PostgreSQL)
 ├─ web/    프론트엔드       (React 19 + Vite + TanStack Router + Tailwind)
 ├─ site/   랜딩 홈페이지    (Next.js 16)
 └─ docs/   공식 문서        (Mintlify, openapi.json 포함)

packages/
 ├─ mcp/            MCP 서버 — Claude / Cursor 에서 Kaneo 조작
 ├─ email/          이메일 템플릿 (React Email + nodemailer)
 ├─ permissions/    권한 관리 로직
 ├─ planka-import/  PLANKA → Kaneo 데이터 마이그레이션 CLI
 └─ typescript-config/  공용 TS 설정

i18n/          19개 언어 번역 (ko-KR 한국어 포함)
charts/        Kubernetes Helm 차트
tests/         유닛 + 통합 테스트
skills/ .claude/ .agents/   AI 에이전트용 스킬 문서
compose.yml    Docker Compose (원클릭 실행)
Dockerfile.kaneo
```

### API 기능 모듈 (`apps/api/src`, 37개)

`activity`, `billing`, `column`, `comment`, `database`, `discord-integration`,
`events`, `external-link`, `generic-webhook-integration`, `gitea-integration`,
`github-integration`, `instance`, `integrations`, `invitation`, `label`, `mcp`,
`notification`, `notification-preferences`, `oauth`, `plugins`, `project`,
`redis`, `scheduler`, `search`, `slack-integration`, `storage`, `task`,
`task-relation`, `telegram-integration`, `time-entry`, `user`, `workflow-rule`,
`workspace`, `ws` 등

---

## 3. 주요 기능

| 분류 | 기능 |
| --- | --- |
| 작업 관리 | 칸반보드, 리스트뷰, 백로그, 캘린더, 간트차트, 서브태스크, 작업 연관관계 |
| 협업 | 워크스페이스 / 팀, 초대, 댓글, 활동 로그, 라벨, 알림 |
| 생산성 | 타임 트래킹, 커맨드 팔레트(⌘K), 전체 검색, 단축키, 대량 선택 |
| 자동화 | `workflow-rule` — 조건 기반 자동 상태 변경 |
| 연동 | GitHub, Gitea, Slack, Discord, Telegram, 범용 웹훅 |
| AI | **MCP 서버 내장** (`/api/mcp`) — Claude / Cursor 에서 태스크 조작 |
| 기타 | OAuth 로그인, S3 파일 업로드, 다국어, 다크모드, 실시간(WebSocket) |

---

## 4. 기술 스택

| 영역 | 기술 |
| --- | --- |
| 백엔드 | Hono, Drizzle ORM, PostgreSQL, better-auth, Zod/OpenAPI, WebSocket, ioredis |
| 프론트엔드 | React 19, Vite, TanStack Router/Query, Tailwind, dnd-kit, Radix/Base UI |
| 인프라 | Turborepo, pnpm, Docker, Helm, Sentry |
| 품질 | Biome (린트/포맷), Vitest, Husky, commitlint, semantic-release |
| 런타임 요구사항 | Node.js >= 24, pnpm 10.x |

---

## 5. 설치 방법

### 방법 A — Docker로 바로 쓰기 (권장, 약 5분)

**0) 준비물**: [Docker Desktop](https://www.docker.com/products/docker-desktop/) 설치 후 실행

**1) 설정 파일 생성**

```bash
cd kaneo
cp .env.sample .env
```

**2) 필수 값 2개 채우기**

암호화 키 생성:

```bash
openssl rand -hex 32
```

Windows PowerShell 대안:

```powershell
-join ((1..64) | ForEach-Object { '{0:x}' -f (Get-Random -Max 16) })
```

`.env` 수정:

```bash
KANEO_CLIENT_URL=http://localhost:5173   # 기본값 그대로
POSTGRES_PASSWORD=원하는_비밀번호
AUTH_SECRET=위에서_생성한_32자_이상_문자열   # 비워두면 재시작 시 세션 소실
```

**3) 실행**

```bash
docker compose up -d
```

**4) 접속** → http://localhost:5173

**운영 명령어**

```bash
docker compose ps        # 상태 확인
docker compose logs -f   # 로그 확인
docker compose stop      # 일시 중지
docker compose start     # 재시작
docker compose down      # 종료 (데이터 유지)
docker compose down -v   # 데이터까지 완전 삭제 (주의)
```

### 방법 B — 개발 모드 (코드 수정 시 즉시 반영)

**0) 준비물**: Node.js 24+ / `npm install -g pnpm`

**1) DB만 Docker로 실행**

```bash
docker compose up -d postgres
```

**2) `.env` 설정**

```bash
KANEO_CLIENT_URL=http://localhost:5173
KANEO_API_URL=http://localhost:1337
POSTGRES_PASSWORD=원하는_비밀번호
AUTH_SECRET=32자_이상_문자열
DATABASE_URL=postgresql://kaneo:원하는_비밀번호@localhost:5432/kaneo
```

> `DATABASE_URL` 안의 비밀번호는 `POSTGRES_PASSWORD` 와 반드시 동일해야 합니다.

**3) 설치 및 실행**

```bash
pnpm install
pnpm dev
```

| 주소 | 설명 |
| --- | --- |
| http://localhost:5173 | 웹 앱 (프론트엔드) |
| http://localhost:1337 | API 서버 (백엔드) |

> 마이그레이션은 API 시작 시 자동 실행됩니다. 실패하면 프로세스가 종료되므로
> 포트가 죽으면 로그를 확인하세요.

---

## 6. 사용법

1. **회원가입 / 로그인** — 최초 가입자가 관리자
2. **워크스페이스 생성** — 회사 / 팀 단위 (예: "우리회사")
3. **프로젝트 생성** — 하나의 일 덩어리 (예: "쇼핑몰 개발")
4. **뷰 선택**
   - **Board**: 상태 기반 실행용 (가장 많이 사용)
   - **List**: 다수 작업을 빠르게 훑을 때
   - **Backlog**: 계획 / 그루밍용
   - 추가로 캘린더, 간트차트 제공
5. **태스크 생성** — 제목은 행동 중심으로. 담당자 / 우선순위(낮음·보통·높음·긴급) / 마감일 / 라벨 설정
6. **드래그로 상태 이동** — `할 일 → 진행 중 → 완료`. WebSocket으로 팀원 화면에 실시간 반영
7. **필터 활용** — 상태 / 우선순위 / 담당자 / 마감일 / 라벨
8. **검색** — `⌘K` (macOS) / `Ctrl+K` (Windows)
9. **팀원 초대** — SMTP 설정 필요 (혼자 쓰면 생략 가능)

```bash
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your@gmail.com
SMTP_PASSWORD=앱_비밀번호
SMTP_FROM=your@gmail.com
```

> SMTP를 설정하면 기본 로그인 방식이 이메일 인증코드(OTP)로 바뀝니다.
> 이메일/비밀번호 방식을 유지하려면 `DISABLE_EMAIL_OTP_SIGN_IN=true`.

10. **알림 연동** — Slack / Discord / Telegram / 범용 웹훅

### AI(MCP) 연결

```bash
npx @kaneo/mcp
```

Claude Desktop / Cursor 중 선택해 등록합니다. 자체 호스팅 주소 지정:

```bash
kaneo-mcp install --target cursor-user -y --api-url http://localhost:1337
```

HTTP 방식은 MCP 클라이언트에 `http://localhost:5173/api/mcp` 를 직접 등록하면 됩니다.

---

## 7. 자주 발생하는 문제

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| 페이지가 열리지 않음 | 기동 중 | 1~2분 대기 후 `docker compose logs -f` |
| `port is already allocated` | 5173 포트 충돌 | 해당 프로그램 종료 또는 `compose.yml` 에서 `"5174:5173"` 으로 변경 |
| 로그인이 자꾸 풀림 | `AUTH_SECRET` 미설정 | 32자 이상 값 지정 |
| DB 연결 실패 | 비밀번호 불일치 | `POSTGRES_PASSWORD` 와 `DATABASE_URL` 일치시키기 |
| CORS / Failed to fetch | URL 설정 불일치 | `KANEO_CLIENT_URL` 이 실제 접속 주소와 같은지 확인, 필요 시 `CORS_ORIGINS` 설정 |
| `.env` 수정 후 반영 안 됨 | 재시작 누락 | `docker compose down && docker compose up -d` |

---

## 8. 수익화 아이디어

### 전제 조건 (실제 코드 확인 결과)

1. **MIT 라이선스** — 수정·판매·서비스 제공 모두 허용. 조건은 `LICENSE` 파일 포함뿐
2. **결제 시스템이 이미 구현되어 있음** (`apps/api/src/billing/`)
   - `personal` / `team` 2개 요금제, 월간 / 연간
   - 14일 무료 체험 (`BILLING_TRIAL_DAYS`)
   - 구독 만료 시 자동 차단 미들웨어 (`require-entitlement-middleware.ts`)
   - 결제사는 Creem, `KANEO_CLOUD=true` + Creem 키가 있을 때만 활성화
3. **한국어 번역 99.6% 완료** — 영어 키 1,694개 중 1,687개 번역, **7개 누락**

### 아이디어 (현실성 순)

| 순위 | 아이디어 | 내용 | 예상 수익 |
| --- | --- | --- | --- |
| 1 | **업종 특화 버전** | 인테리어/건설 현장관리, 영상제작, 병원, 학원, 이커머스 등 좁은 시장 공략 | 월 5~30만원 × 업체수 |
| 2 | **AI 자동화 레이어** | 회의록 → 태스크 자동 생성, 주간보고 자동 작성, 위험 일정 알림 (MCP 기반 확장) | 인당 월 구독 |
| 3 | **구축 대행 / 컨설팅** | 데이터 외부 반출이 불가한 조직(공공·병원·법무·금융) 대상 설치 + 커스터마이징 | 구축 50~200만원 + 유지보수 월 10~30만원 |
| 4 | **한국형 클라우드** | 포트원/토스페이먼츠 결제, 카카오 알림톡, 세금계산서, 국내 리전 | 구독 |
| 5 | **플러그인 판매** | 카카오 알림톡, ERP 연동, 근태 연동, 리포트 확장 (`apps/api/src/plugins/` 구조 활용) | 건별/구독 |
| 6 | **마이그레이션 서비스** | Jira / Trello / 노션 → Kaneo 이관 (`packages/planka-import` 확장) | 건당 100~300만원 |
| 7 | **교육 콘텐츠** | 최신 스택 실전 강의, 유튜브, 전자책 | 부수입 |
| 8 | **커리어** | 업스트림 기여 이력 → 이직 / 연봉 협상 | 간접 |

### 주의사항

- **이름 / 로고는 사용 불가** — 코드는 MIT지만 "Kaneo" 브랜드는 별개. 다른 이름으로 서비스할 것
- **업스트림 속도가 빠름** — 현재 v2.22.0, CHANGELOG 18만 줄. 포크 방치 시 급격히 뒤처짐
- **MIT는 경쟁자에게도 열려 있음** — 코드가 아니라 **서비스와 도메인 지식**으로 차별화해야 함
- **SaaS 운영 부담** — 초기에는 구축 대행(3번)부터 시작하는 편이 안전
- **본가 클라우드가 이미 존재** — 단순 복제형 클라우드는 승산 낮음. 한국화가 전제

### 추천 실행 순서

```
1단계 (~1개월)   설치·사용 + 한국어 누락 7개 번역 PR → 기여 이력 확보 및 코드 파악
2단계 (1~3개월)  카카오 알림톡 플러그인 개발 → 한국 시장 진입 무기 확보
3단계 (3~6개월)  특정 업종 특화 버전 + 구축 대행 1건 수주 → 첫 매출
4단계 (6개월~)   해당 업종 SaaS 전환 + AI 기능 결합 → 구독 모델
```

---

## 9. PHP 전환 검토 결과

### 실측 코드 규모

| 항목 | 수치 |
| --- | --- |
| 백엔드 (`apps/api/src`) | **37,785줄** |
| 프론트엔드 (`apps/web/src`) | **59,400줄** |
| 공용 패키지 (`packages`) | 4,755줄 |
| 전체 소스 파일 | **876개** |
| API 기능 모듈 | **37개** |
| DB 테이블 | **38개** |
| API 엔드포인트 (`openapi.json`) | **111개** |

### 기술 대응표 (TypeScript → PHP)

| 현재 | PHP (Laravel) |
| --- | --- |
| Hono | Laravel |
| Drizzle ORM | Eloquent |
| better-auth | Fortify / Sanctum / Socialite |
| croner (스케줄러) | Laravel Scheduler + Queue |
| WebSocket + Redis Pub/Sub | Laravel Reverb (또는 Soketi / Pusher) |
| Vitest | Pest / PHPUnit |

→ **기술적으로 막히는 부분은 거의 없음.**

### 두 가지 함정

1. **"PHP만으로"는 불가능** — 칸반 드래그앤드롭, 실시간 화면 갱신은 브라우저에서
   동작하는 영역이라 JavaScript가 필수. PHP는 서버 측만 담당 가능
2. **10만 줄 재작성 = 최소 1~2년** — 그 사이 업스트림은 계속 전진

### 3가지 시나리오

**A. 전면 PHP 재작성** — 비권장. 얻는 것이 "PHP로 되어 있음" 뿐.
단, 기능을 대폭 축소한 MVP(칸반 + 로그인)라면 Laravel로 2~3주 수준 가능

**B. 하이브리드 (권장)** — Kaneo는 그대로 두고 PHP는 한국 특화 레이어만 담당

```
   ┌─────────────────┐         ┌──────────────────┐
   │  Kaneo (그대로) │ ──웹훅→ │  PHP (Laravel)   │
   │  칸반 / 실시간  │ ←API──  │  카카오 알림톡   │
   └─────────────────┘         │  포트원 결제     │
                               │  세금계산서      │
                               │  관리자 / 통계   │
                               │  고객사 포털     │
                               └──────────────────┘
```

- 붙이기 쉬운 근거: **문서화된 API 111개** (`apps/docs/openapi.json`),
  **범용 웹훅 플러그인** (`apps/api/src/plugins/generic-webhook`)
- 어려운 부분(칸반·실시간·인증)은 그대로 활용, PHP가 강한 영역(결제·알림·관리자)만 개발
- 첫 결과물까지 2~3주, 업스트림 업데이트도 그대로 수용 가능

**C. PHP 오픈소스에서 새로 시작** — Kaneo를 포팅하지 않고 PHP 기반 오픈소스를 베이스로 채택.
단 **라이선스 검증 필수** (AGPL 계열은 SaaS 제공 시 소스 공개 의무가 발생할 수 있어 사업에 치명적)

### 결론

**시나리오 B(하이브리드)를 권장.**
첫 PHP 산출물은 **"카카오 알림톡 중계기"** 가 적합 — 약 200줄 규모, Kaneo 코드를
전혀 수정하지 않으며, 한국 고객 대상 세일즈 포인트로서 효과가 가장 큼.

```php
// Kaneo 웹훅 수신 → 알림톡 발송 (개념 예시)
Route::post('/kaneo-webhook', function (Request $r) {
    $task = $r->input('task');
    Alimtalk::send(
        to: $task['assignee']['phone'],
        template: 'TASK_ASSIGNED',
        vars: ['title' => $task['title']]
    );
});
```

---

## 10. 다음 액션 후보

- [ ] 로컬에 Docker로 띄우고 실제 사용해보기
- [ ] `i18n/ko-KR.json` 누락 7개 키 번역 → 업스트림 PR
- [ ] Laravel 프로젝트 뼈대 + Kaneo 웹훅 수신 엔드포인트 구현
- [ ] 타겟 업종 확정 후 필요한 커스텀 기능 설계
- [ ] PHP 오픈소스 후보 라이선스 비교 조사 (시나리오 C 선택 시)

---

## 참고 문서 (저장소 내부)

| 파일 | 내용 |
| --- | --- |
| `README.md` | 프로젝트 개요, 배포 방법 |
| `ENVIRONMENT_SETUP.md` | 환경변수 전체 목록, CORS 등 트러블슈팅 |
| `CONTRIBUTING.md` | 기여 가이드 |
| `AGENTS.md` | AI 에이전트 작업 규칙 |
| `apps/docs/openapi.json` | API 스펙 (111개 엔드포인트) |
| `charts/kaneo/README.md` | Kubernetes Helm 차트 가이드 |
| `.claude/skills/verify/SKILL.md` | 로컬 검증 레시피 |
