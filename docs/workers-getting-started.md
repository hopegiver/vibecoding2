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
├── .claude/
│   └── prompts.md                 # AI 개발 가이드라인
├── src/
│   ├── index.js                   # 앱 초기화 및 미들웨어
│   ├── openapi.js                 # OpenAPI 스펙
│   ├── routes/                    # HTTP 라우트 핸들러
│   │   ├── auth.js                # 인증 (로그인)
│   │   └── users.js               # 사용자 관리
│   ├── services/                  # 비즈니스 로직
│   │   ├── authService.js         # 인증 서비스
│   │   ├── userService.js         # 사용자 서비스
│   │   └── openaiService.js       # OpenAI 연동
│   ├── middleware/                 # 미들웨어
│   │   ├── auth.js                # JWT 인증
│   │   └── errorHandler.js        # 에러 처리
│   └── utils/
│       └── utils.js               # 유틸리티 함수
├── wrangler.toml                  # Workers 설정
├── package.json
├── schema.sql                     # D1 데이터베이스 스키마
├── CONTRIBUTING.md                # 개발 규칙 (필독!)
└── README.md
```

### .claude/prompts.md

템플릿에는 Claude Code를 위한 가이드라인이 포함되어 있습니다:
- 코드 작성 전 `CONTRIBUTING.md` 필독
- camelCase 파일명, PascalCase 클래스명
- Service 기반 아키텍처 (env 주입)
- 기존 코드 패턴 참고

### CONTRIBUTING.md (개발 규칙)

프로젝트의 핵심 개발 규칙이 정리되어 있습니다:
- 아키텍처 패턴 (routes → services → utils)
- 코딩 컨벤션 (네이밍, 내보내기 규칙)
- Cloudflare 바인딩 사용법 (KV, D1, R2)
- 인증 패턴 (JWT, PUBLIC_PATHS)
- 에러 처리 규칙

## 2. 기술 스택

| 구성 요소 | 버전 | 용도 |
|-----------|------|------|
| Hono | ^4.0.0 | 웹 프레임워크 |
| jose | ^5.2.0 | JWT 인증 |
| @hono/swagger-ui | ^0.2.0 | API 문서 |
| Wrangler | ^3.0.0 | CLI 및 개발 서버 |

## 3. wrangler.toml

```toml
name = "myworker"
main = "src/index.js"
compatibility_date = "2024-01-01"

[vars]
ENVIRONMENT = "development"

# 필요 시 주석 해제
# [[kv_namespaces]]
# binding = "CACHE"
# id = "xxx"

# [[d1_databases]]
# binding = "DB"
# database_name = "myworker-db"
# database_id = "xxx"

# [[r2_buckets]]
# binding = "STORAGE"
# bucket_name = "myworker-storage"

[env.dev]
vars = { ENVIRONMENT = "development" }

[env.production]
vars = { ENVIRONMENT = "production" }
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

## 5. 프로젝트 커스터마이즈

### 기존 라우트 수정

`src/routes/`의 auth.js, users.js를 프로젝트에 맞게 변경.

### 새 기능 추가 순서

1. `src/routes/`에 라우트 파일 생성
2. `src/services/`에 서비스 클래스 생성
3. `src/index.js`에 라우트 등록
4. `src/openapi.js`에 API 문서 추가

> 라우팅 및 서비스 패턴은 [라우팅](workers-routing.md) 참고.

## 체크리스트

- [ ] workers-template 클론 완료
- [ ] npm install 완료
- [ ] .dev.vars 생성 (JWT_SECRET)
- [ ] CONTRIBUTING.md 숙지
- [ ] 로컬 서버 작동 확인 (`npm run dev`)
- [ ] Swagger 문서 확인 (`/docs`)

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
