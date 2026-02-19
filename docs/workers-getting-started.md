# Cloudflare Workers - 프로젝트 시작하기

Cloudflare Workers + Hono 프레임워크 기반 API 프로젝트를 시작하는 방법을 안내합니다.

## 전제조건

- Node.js 18 이상
- Cloudflare 계정
- VSCode + Claude Code

## 1. workers-template으로 시작하기

### 템플릿 클론

```bash
# workers-template 클론
git clone https://github.com/hopegiver/workers-template myworker
cd myworker

# 기존 git 히스토리 제거 후 새로 초기화
rm -rf .git
git init

# 의존성 설치
npm install
```

### 템플릿 기본 구조

```
myworker/
├── CLAUDE.md                      # AI 개발 가이드 (핵심 규칙)
├── .claude/
│   ├── rules/
│   │   └── architecture.md        # 아키텍처 규칙 (자동 적용)
│   ├── commands/                  # 슬래시 커맨드
│   │   ├── endpoint.md            # /endpoint - 엔드포인트 추가
│   │   ├── service.md             # /service - 서비스 생성
│   │   ├── review.md              # /review - 코드 리뷰
│   │   └── test.md                # /test - 테스트 작성
│   └── templates/                 # 코드 생성 템플릿
│       ├── route.md               # 라우트 CRUD 템플릿
│       ├── service.md             # 서비스 클래스 템플릿
│       └── test.md                # 테스트 템플릿
├── docs/                          # 세부 개발 가이드
│   ├── project-structure.md
│   ├── coding-conventions.md
│   ├── architecture.md
│   ├── cloudflare-bindings.md
│   ├── authentication.md
│   ├── environment.md
│   ├── error-handling.md
│   └── adding-features.md
├── src/
│   ├── index.js                   # 앱 초기화 (미들웨어, 라우트 등록)
│   ├── openapi.js                 # OpenAPI 3.0 스펙
│   ├── routes/                    # HTTP 라우트 핸들러
│   │   ├── auth.js                # 인증 (로그인)
│   │   └── users.js               # 사용자 관리
│   ├── services/                  # 비즈니스 로직 (클래스)
│   │   ├── authService.js         # 인증 서비스
│   │   ├── userService.js         # 사용자 서비스
│   │   └── openaiService.js       # OpenAI 연동
│   ├── middleware/                 # 미들웨어
│   │   ├── auth.js                # JWT 인증
│   │   └── errorHandler.js        # 에러 처리
│   └── utils/
│       └── utils.js               # 유틸리티 함수
├── test/                          # 테스트
│   ├── routes/
│   │   └── auth.test.js
│   ├── services/
│   │   └── authService.test.js
│   └── utils/
│       └── utils.test.js
├── wrangler.toml                  # Workers 설정
├── package.json
├── vitest.config.js               # 테스트 설정
├── schema.sql                     # D1 데이터베이스 스키마
└── README.md
```

### CLAUDE.md

Claude Code가 자동으로 참조하는 프로젝트 가이드입니다:
- 프로젝트 구조 및 기술 스택
- 주요 명령어 (`npm run dev`, `npm run deploy`, `npm run test`)
- 새 기능 추가 순서 (서비스 → 라우트 → 등록 → OpenAPI)
- 슬래시 커맨드 (`/endpoint`, `/service`, `/review`, `/test`)
- 세부 규칙은 `docs/` 폴더 참조

### .claude/ 폴더

Claude Code가 자동으로 활용하는 설정 파일들입니다:

**rules/architecture.md** - 아키텍처 규칙 (대화 시 자동 적용):
- Request → Middleware → Route → Service → Response 흐름
- 서비스 클래스 패턴 (`constructor(env)`)
- 에러 throw 패턴 (ValidationError→400, UnauthorizedError→401)
- `process.env` 사용 금지, `c.env` / `this.env` 사용

**commands/** - 슬래시 커맨드 (`/endpoint`, `/service`, `/review`, `/test`):
- `/endpoint` - 서비스 + 라우트 + 등록 + OpenAPI 한번에 생성
- `/service` - 서비스 클래스 생성
- `/review` - 프로젝트 컨벤션 기준 코드 리뷰
- `/test` - 지정 기능에 대한 테스트 작성

**templates/** - 코드 생성 시 참조하는 표준 템플릿:
- route.md: CRUD 라우트 패턴 (Hono 라우터, 입력 검증, 인증 정보 접근)
- service.md: 서비스 클래스 패턴 (D1 쿼리, KV 캐시 적용)
- test.md: 테스트 패턴 (라우트 통합 테스트, 서비스 단위 테스트)

### docs/ (세부 개발 가이드)

| 파일 | 내용 |
|------|------|
| project-structure.md | 폴더 구조, 파일별 역할 |
| coding-conventions.md | 네이밍, Export 패턴, 코드 스타일 |
| architecture.md | 레이어 구조, 서비스/라우트 패턴 |
| cloudflare-bindings.md | KV, D1, R2 사용법 |
| authentication.md | JWT, 공개 경로, 사용자 정보 접근 |
| environment.md | .dev.vars, Wrangler Secrets |
| error-handling.md | 에러 throw 패턴, 상태코드 매핑 |
| adding-features.md | 엔드포인트/서비스 추가 순서 |

## 2. 기술 스택

| 구성 요소 | 버전 | 용도 |
|-----------|------|------|
| Hono | ^4.0.0 | 웹 프레임워크 |
| jose | ^5.2.0 | JWT 인증 |
| @hono/swagger-ui | ^0.2.0 | API 문서 |
| Wrangler | ^3.0.0 | CLI 및 개발 서버 |
| Vitest | ~3.2.0 | 테스트 프레임워크 |

## 3. wrangler.toml

```toml
name = "workers-template"
main = "src/index.js"
compatibility_date = "2024-01-01"

[vars]
ENVIRONMENT = "development"

[env.dev]
name = "workers-template-dev"
vars = { ENVIRONMENT = "development" }

[env.production]
name = "workers-template-prod"
vars = { ENVIRONMENT = "production" }

# 필요 시 주석 해제
# [[kv_namespaces]]
# binding = "KV"
# id = "your-kv-namespace-id"

# [[d1_databases]]
# binding = "DB"
# database_name = "your-database"
# database_id = "your-database-id"

# [[r2_buckets]]
# binding = "BUCKET"
# bucket_name = "your-bucket"
```

## 4. 로컬 개발 서버 실행

```bash
npm run dev
```

**접속:**
- API: `http://localhost:8787`
- Swagger 문서: `http://localhost:8787/docs`
- Health 체크: `http://localhost:8787/health`

**테스트 계정:**
- admin / admin123 (role: admin)
- user / user123 (role: user)

### 환경 변수 (.dev.vars)

로컬 개발 시 `.dev.vars` 파일 생성:

```
JWT_SECRET=your-dev-secret-key
```

> Wrangler가 `.dev.vars`를 자동 로드합니다.

### 테스트 실행

```bash
npm run test          # 단일 실행
npm run test:watch    # 감시 모드
```

## 5. 프로젝트 커스터마이즈

### 기존 라우트 수정

`src/routes/`의 auth.js, users.js를 프로젝트에 맞게 변경.

### 새 기능 추가 순서

1. `src/services/`에 서비스 클래스 생성
2. `src/routes/`에 라우트 핸들러 생성 (Hono 라우터 default export)
3. `src/index.js`에 `app.route()` 등록
4. `src/openapi.js`에 API 스펙 업데이트

> 라우팅 및 서비스 패턴은 [라우팅](workers-routing.md) 참고.

## 체크리스트

- [ ] workers-template 클론 완료
- [ ] npm install 완료
- [ ] .dev.vars 생성 (JWT_SECRET)
- [ ] CLAUDE.md 및 docs/ 숙지
- [ ] 로컬 서버 작동 확인 (`npm run dev`)
- [ ] Swagger 문서 확인 (`/docs`)
- [ ] 테스트 통과 확인 (`npm run test`)

## 문제 해결

### 로컬 서버 실행 실패

**증상:** `Error: Could not resolve "src/index.js"`

**해결:**
- wrangler.toml의 `main` 경로 확인
- `npm install` 실행 여부 확인

### JWT 인증 오류

**증상:** `401 Unauthorized`

**해결:**
- `.dev.vars`에 JWT_SECRET 설정 여부 확인
- Authorization 헤더 형식: `Bearer <token>`

### 배포 실패

**증상:** `Error: Authentication error`

**해결:**
- `wrangler login` 재실행
- Cloudflare 계정 권한 확인

## 관련 문서

- [라우팅](workers-routing.md)
- [배포](workers-deployment.md)
- [프로젝트 구조 표준](project-structure.md)
