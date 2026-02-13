# 프로젝트 구조 표준

## 개요

일관된 프로젝트 구조는 Claude Code가 코드를 더 정확하게 이해하고 생성하는 데 도움이 됩니다. 이 문서는 **플랫폼별 권장 구조**를 제시합니다.

## 공통 구조

모든 프로젝트에 공통으로 적용되는 기본 구조:

```
project-root/
├── .claude/                # Claude Code 설정
│   ├── rules/             # 개발 규칙
│   ├── commands/          # 슬래시 커맨드
│   ├── templates/         # 코드 템플릿
│   └── memory/            # 작업 이력 (선택)
├── CLAUDE.md              # 프로젝트 컨텍스트
├── .gitignore
├── README.md
└── [플랫폼별 구조]
```

### .claude 폴더 상세

```
.claude/
├── rules/
│   ├── architecture.md         # 아키텍처 패턴
│   ├── coding-conventions.md   # 코딩 컨벤션
│   ├── style-guide.md          # 스타일 가이드
│   └── security.md             # 보안 규칙
├── commands/
│   ├── crud.md                # CRUD 생성 커맨드
│   ├── page.md                # 페이지 생성 커맨드
│   └── review.md              # 코드 리뷰 커맨드
├── templates/
│   ├── crud-list.md           # 목록 페이지 템플릿
│   ├── crud-insert.md         # 등록 페이지 템플릿
│   ├── crud-modify.md         # 수정 페이지 템플릿
│   └── api-endpoint.md        # API 엔드포인트 템플릿
└── memory/ (선택)
    ├── status.md              # 현재 작업 상태
    ├── history.md             # 작업 이력
    └── next-tasks.md          # 다음 할 일
```

---

## 1. Cloudflare Workers 구조

### 기본 구조

```
workers-project/
├── .claude/
│   ├── rules/
│   │   ├── architecture.md
│   │   └── coding-conventions.md
│   ├── commands/
│   │   └── crud.md
│   └── templates/
│       └── api-endpoint.md
├── CLAUDE.md
├── wrangler.toml
├── package.json
├── src/
│   ├── index.js              # 엔트리 포인트
│   ├── routes/               # API 라우트
│   │   ├── auth.js
│   │   ├── users.js
│   │   └── products.js
│   ├── services/             # 비즈니스 로직 (클래스)
│   │   ├── authService.js
│   │   ├── userService.js
│   │   └── productService.js
│   ├── middleware/           # 미들웨어
│   │   ├── auth.js
│   │   ├── cors.js
│   │   └── errorHandler.js
│   └── utils/                # 유틸리티 함수
│       ├── jwt.js
│       └── validators.js
├── migrations/               # D1 마이그레이션
│   ├── 0001_create_users.sql
│   └── 0002_create_products.sql
└── tests/                    # 테스트
    ├── routes/
    └── services/
```

### 파일 명명 규칙

- **서비스**: `camelCase` (예: `userService.js`)
- **라우트**: `camelCase` (예: `users.js`)
- **클래스**: `PascalCase` (예: `class UserService`)
- **상수**: `UPPER_SNAKE_CASE`

### wrangler.toml 예제

```toml
name = "my-api"
main = "src/index.js"
compatibility_date = "2024-01-01"

[env.production]
name = "my-api-prod"

[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "xxx"

[[kv_namespaces]]
binding = "KV"
id = "xxx"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "my-bucket"
```

---

## 2. Cloudflare Pages 구조

### 기본 구조 (ViewLogic 패턴)

```
pages-project/
├── .claude/
│   ├── rules/
│   │   ├── viewlogic-guide.md
│   │   └── style-guide.md
│   ├── commands/
│   │   └── page.md
│   └── templates/
│       └── pages-viewlogic.md
├── CLAUDE.md
├── index.html
├── favicon.ico
├── src/
│   ├── views/               # HTML 템플릿 (CSS 금지!)
│   │   ├── layout/          # 레이아웃 템플릿
│   │   │   └── default.html
│   │   ├── goals/
│   │   │   ├── my-goals.html
│   │   │   └── detail.html
│   │   └── team/
│   │       └── tasks.html
│   ├── logic/               # JavaScript 로직
│   │   ├── layout/          # 레이아웃 스크립트
│   │   │   └── default.js
│   │   ├── goals/
│   │   │   ├── my-goals.js
│   │   │   └── detail.js
│   │   └── team/
│   │       └── tasks.js
│   └── components/          # 재사용 컴포넌트 (선택)
├── css/
│   └── base.css             # 커스텀 스타일 (Bootstrap 보완)
├── images/
└── functions/               # Pages Functions (API)
    └── api/
        └── [[path]].js
```

### 파일 명명 규칙

- **파일명 = 라우트**: `goals/my-goals.html` → `#/goals/my-goals`
- **HTML 파일**: `kebab-case` (예: `my-goals.html`)
- **JS 파일**: `kebab-case` (예: `my-goals.js`)
- **컴포넌트**: `PascalCase` (예: `TaskCard.vue`)

### 핵심 원칙

✅ **HTML과 JS 완전 분리**
- `views/` - HTML만 (CSS 금지!)
- `logic/` - JavaScript만
- `css/` - CSS만

❌ **절대 금지**
- HTML 파일에 `<style>` 태그
- HTML 파일에 `<script>` 태그
- Logic 파일에 HTML 문자열

---

## 3. 맑은프레임워크 구조

### 기본 구조

```
malgn-project/
├── .claude/
│   ├── settings.json        # 권한 및 훅 설정
│   ├── rules/
│   │   └── malgn.md         # 코딩 규칙 (MCP 참조)
│   ├── commands/
│   │   ├── crud.md          # CRUD 생성 커맨드
│   │   ├── api.md           # API 생성 커맨드
│   │   ├── new-page.md      # 페이지 생성 커맨드
│   │   ├── schema.md        # 스키마 생성 커맨드
│   │   ├── validate.md      # 코드 검증 커맨드
│   │   └── review.md        # 코드 리뷰 커맨드
│   └── hooks/
│       └── post-write.sh    # 자동 검증 훅
├── .mcp.json                # MCP 서버 설정
├── CLAUDE.md                # 프로젝트 컨텍스트
├── GUIDE.md                 # 코딩 가이드 (상세)
├── build.xml                # Ant 빌드 스크립트
├── schema.sql               # DB 스키마
├── src/
│   └── dao/                 # DAO 클래스 (Java)
│       ├── UserDao.java
│       ├── BoardDao.java
│       └── ApplyDao.java
└── public_html/
    ├── init.jsp             # 공통 초기화
    ├── index.jsp            # 루트 리다이렉트
    ├── main/                # 메인 모듈
    ├── member/              # 회원 모듈
    ├── board/               # 게시판 모듈
    ├── admin/               # 관리자 모듈
    ├── html/                # HTML 템플릿
    │   ├── layout/          # 레이아웃 (layout_xxx.html)
    │   ├── main/
    │   ├── member/
    │   └── board/
    ├── api/                 # REST API
    │   ├── init.jsp         # API 초기화 (JWT, CORS)
    │   └── index.jsp        # API 라우터
    ├── assets/              # 정적 파일
    │   ├── css/
    │   └── js/common.js
    └── WEB-INF/
        ├── config.xml       # DB 설정 (JNDI)
        ├── web.xml          # 서블릿 매핑 (/api/*)
        ├── lib/malgn.jar    # 프레임워크 라이브러리
        └── classes/         # 컴파일된 DAO
```

### 파일 명명 규칙

- **JSP 파일**: `snake_case` (예: `user_list.jsp`)
- **HTML 파일**: `snake_case` (예: `user_list.html`)
- **DAO 클래스**: `PascalCase` (예: `UserDao.java`)
- **테이블**: `tb_` 접두사 (예: `tb_user`, `tb_board`)
- **레이아웃**: `layout_` 접두사 (예: `layout_main.html`)

### 핵심 원칙

- JSP와 HTML 완전 분리 (JSP: 로직, HTML: 출력)
- MCP 도구로 규칙/패턴/클래스 참조
- try-catch 금지, boolean 리턴 사용
- POST 처리 후 return 필수
- DAO 수정 시 `ant compile` 필수

---

## 환경별 설정

### 개발/프로덕션 분리

```
project-root/
├── .env                     # Git 제외!
├── .env.example             # Git 포함
├── .env.development
├── .env.production
└── config/
    ├── development.js
    └── production.js
```

### .env 예제

```bash
# .env (로컬 개발)
DATABASE_URL=sqlite://local.db
API_URL=http://localhost:8787
DEBUG=true

# .env.production (프로덕션)
DATABASE_URL=postgresql://prod.db
API_URL=https://api.example.com
DEBUG=false
```

---

## 프로젝트 초기화 체크리스트

새 프로젝트 시작 시:

- [ ] `.claude/` 폴더 생성
- [ ] `CLAUDE.md` 작성
- [ ] `.gitignore` 설정
- [ ] `.env.example` 생성
- [ ] 플랫폼별 기본 구조 생성
- [ ] `README.md` 작성

---

## Claude Code 활용 팁

### 구조 기반 프롬프트

```
프롬프트: "현재 프로젝트 구조를 분석하고,
src/routes/products.js와 src/services/productService.js를 생성해줘.
기존 users 모듈과 동일한 패턴으로."
```

Claude Code가 프로젝트 구조를 이해하고 일관된 코드를 생성합니다.

### 구조 리팩토링

```
프롬프트: "현재 레이어별 구조를 기능별 구조로 리팩토링해줘.
auth, users, products 모듈로 분리."
```

---

## 다음 단계

- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트
- [첫 프로젝트 만들기](first-project.md) - 실전 프로젝트 생성

---

[← 목차로 돌아가기](../_sidebar.md)
