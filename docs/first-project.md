# 첫 프로젝트 만들기

## 개요

Claude Code로 **실전 CRUD 프로젝트**를 처음부터 만들어봅니다. 이 가이드를 따라하면 약 **15-20분** 안에 작동하는 API 서버를 구축할 수 있습니다.

## 프로젝트 선택

아래 3가지 중 하나를 선택하세요:

1. **맑은프레임워크** - Java/JSP 기반 웹 애플리케이션
2. **Cloudflare Pages** - Vue.js 기반 프론트엔드
3. **Cloudflare Workers** - Hono 기반 REST API

이 가이드는 **Cloudflare Workers**를 예제로 진행합니다.

---

## Step 1: 프로젝트 초기화 (2분)

### 1-1. 프로젝트 폴더 생성

```bash
mkdir my-first-api
cd my-first-api
```

### 1-2. package.json 생성

```bash
npm init -y
```

### 1-3. 의존성 설치

```bash
npm install hono
npm install -D wrangler
```

### 1-4. wrangler.toml 생성

```toml
name = "my-first-api"
main = "src/index.js"
compatibility_date = "2024-01-01"

[[d1_databases]]
binding = "DB"
database_name = "my-first-db"
database_id = "local"

[[kv_namespaces]]
binding = "KV"
id = "local"
```

---

## Step 2: .claude 폴더 설정 (3분)

### 2-1. 폴더 구조 생성

```bash
mkdir -p .claude/rules
mkdir -p .claude/templates
```

### 2-2. CLAUDE.md 생성

**파일: `CLAUDE.md`**

```markdown
# My First API

## 프로젝트 개요
Cloudflare Workers 기반 REST API 서버. 사용자(User) 관리 기능 제공.

기술 스택: Hono + D1 + KV

## 프로젝트 구조
\`\`\`
src/
├── routes/          # API 라우트
├── services/        # 비즈니스 로직 (클래스 기반)
└── index.js         # 엔트리 포인트
\`\`\`

## 핵심 규칙
- 서비스는 클래스 기반으로 작성
- 라우트에는 비즈니스 로직 금지
- 파일명은 camelCase
- KV 캐시 우선 사용 (TTL: 1시간)

## 현재 작업
- 사용자 CRUD API 구현 중
```

### 2-3. .claude/rules/architecture.md 생성

**파일: `.claude/rules/architecture.md`**

```markdown
# Workers 아키텍처 규칙

## 필수 사항
- 레이어 구조: Request → Route → Service → D1/KV → Response
- 서비스는 클래스 기반으로 작성
- KV 캐시 우선 사용, D1은 백업
- 비동기 함수는 async/await 사용

## 금지 사항
- 라우트에서 직접 DB 접근 금지
- Promise then/catch 사용 금지
- 비즈니스 로직을 라우트에 작성 금지
```

---

## Step 3: Claude Code로 코드 생성 (5분)

이제 Claude Code를 사용하여 코드를 생성합니다!

### 3-1. VSCode에서 프로젝트 열기

```bash
code .
```

### 3-2. Claude Code 채팅 열기

- `Ctrl+Shift+P` (macOS: `Cmd+Shift+P`)
- "Claude Code: Open Chat" 입력

### 3-3. 프롬프트 1: 엔트리 포인트 생성

```
프롬프트: "CLAUDE.md와 .claude/rules/architecture.md를 읽고,
Hono 기반 엔트리 포인트 (src/index.js)를 만들어줘.
- Hono 앱 초기화
- 에러 핸들링 미들웨어 포함
- /health 엔드포인트 추가"
```

**생성될 파일:**
- `src/index.js`

### 3-4. 프롬프트 2: User CRUD API 생성

```
프롬프트: ".claude/rules/architecture.md의 패턴을 따라서
User CRUD API를 만들어줘.

요구사항:
- UserService 클래스 (services/userService.js)
- users 라우트 (routes/users.js)
- GET /users - 목록 조회
- GET /users/:id - 상세 조회
- POST /users - 생성
- PUT /users/:id - 수정
- DELETE /users/:id - 삭제

KV 캐시를 우선 사용하고, D1을 백업으로 사용해줘."
```

**생성될 파일:**
- `src/services/userService.js`
- `src/routes/users.js`

**업데이트될 파일:**
- `src/index.js` (라우트 등록)

### 3-5. 프롬프트 3: D1 스키마 생성

```
프롬프트: "users 테이블을 위한 D1 마이그레이션 파일을 만들어줘.

컬럼:
- id (INTEGER PRIMARY KEY AUTOINCREMENT)
- name (TEXT NOT NULL)
- email (TEXT UNIQUE NOT NULL)
- created_at (TEXT NOT NULL)
- updated_at (TEXT)"
```

**생성될 파일:**
- `migrations/0001_create_users_table.sql`

---

## Step 4: 로컬 테스트 (3분)

### 4-1. D1 데이터베이스 생성 (로컬)

```bash
npx wrangler d1 execute my-first-db --local --file=migrations/0001_create_users_table.sql
```

### 4-2. 개발 서버 실행

```bash
npx wrangler dev
```

### 4-3. API 테스트

**POST 요청 (사용자 생성):**
```bash
curl -X POST http://localhost:8787/users \
  -H "Content-Type: application/json" \
  -d '{"name":"홍길동","email":"hong@example.com"}'
```

**GET 요청 (목록 조회):**
```bash
curl http://localhost:8787/users
```

**GET 요청 (상세 조회):**
```bash
curl http://localhost:8787/users/1
```

---

## Step 5: 체크포인트 생성 (2분)

작업 내용을 정리하고 체크포인트를 만듭니다.

### 5-1. 프롬프트: 체크포인트 생성

```
프롬프트: "지금까지 작업한 내용을 요약해서 CHECKPOINT.md 파일로 저장해줘."
```

**생성될 파일: `CHECKPOINT.md`**

```markdown
# 체크포인트 - 2024-01-29 15:30

## 완료한 작업
- ✅ Hono 기반 엔트리 포인트 생성
- ✅ UserService 클래스 구현 (KV 캐시 포함)
- ✅ User CRUD API 구현 (5개 엔드포인트)
- ✅ D1 마이그레이션 파일 생성
- ✅ 로컬 테스트 완료

## 생성된 파일
- src/index.js
- src/services/userService.js
- src/routes/users.js
- migrations/0001_create_users_table.sql

## 다음 작업
- [ ] 인증 미들웨어 추가
- [ ] 유효성 검증 추가
- [ ] 에러 메시지 국제화
- [ ] 테스트 작성
```

---

## Step 6: Git 커밋 (2분)

### 6-1. .gitignore 생성

```
프롬프트: "Node.js와 Wrangler를 위한 .gitignore 파일을 만들어줘."
```

### 6-2. Git 초기화 및 커밋

```bash
git init
git add .
git commit -m "Initial commit: User CRUD API"
```

---

## Step 7: 배포 (선택, 3분)

### 7-1. Cloudflare 계정 로그인

```bash
npx wrangler login
```

### 7-2. D1 데이터베이스 생성 (프로덕션)

```bash
npx wrangler d1 create my-first-db
```

출력된 `database_id`를 `wrangler.toml`에 업데이트합니다.

### 7-3. 마이그레이션 실행

```bash
npx wrangler d1 execute my-first-db --file=migrations/0001_create_users_table.sql
```

### 7-4. Workers 배포

```bash
npx wrangler deploy
```

배포 완료! URL이 출력됩니다: `https://my-first-api.your-subdomain.workers.dev`

---

## 학습 포인트

이 과정에서 배운 것:

1. ✅ **Claude Code 활용법**
   - CLAUDE.md로 프로젝트 컨텍스트 제공
   - .claude/rules로 코딩 규칙 정의
   - 명확한 프롬프트 작성

2. ✅ **바이브코딩 워크플로우**
   - 요구사항 → 프롬프트 → 코드 생성 → 테스트
   - 체크포인트 생성으로 진행 상황 관리

3. ✅ **Cloudflare Workers 아키텍처**
   - Hono 프레임워크
   - 서비스 레이어 패턴
   - KV 캐시 + D1 데이터베이스

---

## 다음 단계

### 기능 확장 프롬프트 예제

**인증 추가:**
```
프롬프트: "JWT 기반 인증 미들웨어를 추가해줘.
- POST /auth/login (이메일/비밀번호)
- JWT 토큰 발급 (유효기간 7일)
- 인증이 필요한 엔드포인트에 미들웨어 적용"
```

**유효성 검증:**
```
프롬프트: "Zod를 사용해서 User API 요청 데이터 검증을 추가해줘.
- 이메일 형식 검증
- 이름 최소 2자 이상
- 중복 이메일 체크"
```

**페이징:**
```
프롬프트: "GET /users에 페이징 기능을 추가해줘.
- ?page=1&limit=20
- 응답에 total, page, limit 포함"
```

### 더 알아보기

- [컨텍스트 관리 및 체크포인트](context-management.md) - 장기 프로젝트 관리
- [Workers 실전 가이드](workers-getting-started.md) - 고급 Workers 기능
- [.claude/templates 활용법](claude-templates.md) - 재사용 가능한 템플릿 만들기

---

## 다른 플랫폼으로 시작하기

### Cloudflare Pages

- [Pages 프로젝트 시작하기](pages-getting-started.md)
- [Pages .claude 설정](pages-claude-setup.md)

### 맑은프레임워크

- [맑은프레임워크 시작하기](malgn-getting-started.md)
- [맑은프레임워크 .claude 설정](malgn-claude-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
