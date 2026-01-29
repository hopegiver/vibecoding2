# Cloudflare Pages - 환경 변수 및 설정

환경 변수, 바인딩, 설정 파일을 관리하는 방법을 학습합니다.

## 환경 변수 기본

### Cloudflare 대시보드에서 설정

1. **Pages 프로젝트 선택**
2. **Settings → Environment variables**
3. **Add variable**

**예시:**
```
Variable name: API_KEY
Value: secret_key_123
Environment: Production
```

### Functions에서 사용

`functions/api/example.js`:

```javascript
export async function onRequestGet(context) {
    const { env } = context;

    // 환경 변수 사용
    const apiKey = env.API_KEY;
    const dbUrl = env.DATABASE_URL;

    return new Response(JSON.stringify({
        message: 'API Key loaded'
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## wrangler.toml 설정

### 기본 구조

`wrangler.toml`:

```toml
name = "myapp"
compatibility_date = "2024-01-01"

# Pages 설정
pages_build_output_dir = "/"

# 환경 변수 (개발용)
[vars]
API_URL = "http://localhost:8787"
DEBUG = "true"

# D1 데이터베이스 바인딩
[[d1_databases]]
binding = "DB"
database_name = "myapp-db"
database_id = "xxx-xxx-xxx"

# KV 네임스페이스 바인딩
[[kv_namespaces]]
binding = "CACHE"
id = "xxx-xxx-xxx"
preview_id = "yyy-yyy-yyy"

# R2 버킷 바인딩
[[r2_buckets]]
binding = "UPLOADS"
bucket_name = "myapp-uploads"
preview_bucket_name = "myapp-uploads-preview"

# Durable Objects 바인딩
[[durable_objects.bindings]]
name = "COUNTER"
class_name = "Counter"
script_name = "counter-worker"
```

### 환경별 설정

**프로덕션:**

```toml
[env.production]
vars = { API_URL = "https://api.myapp.com", DEBUG = "false" }

[[env.production.d1_databases]]
binding = "DB"
database_id = "prod-xxx-xxx"
```

**스테이징:**

```toml
[env.staging]
vars = { API_URL = "https://staging-api.myapp.com", DEBUG = "true" }

[[env.staging.d1_databases]]
binding = "DB"
database_id = "staging-xxx-xxx"
```

## D1 데이터베이스 바인딩

### D1 생성

```bash
# D1 데이터베이스 생성
npx wrangler d1 create myapp-db

# 출력 예시:
# database_id = "xxx-xxx-xxx"
```

### wrangler.toml에 추가

```toml
[[d1_databases]]
binding = "DB"
database_name = "myapp-db"
database_id = "xxx-xxx-xxx"
```

### 스키마 생성

`schema.sql`:

```sql
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    created_at TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    created_at TEXT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**스키마 적용:**

```bash
# 로컬
npx wrangler d1 execute myapp-db --local --file=schema.sql

# 프로덕션
npx wrangler d1 execute myapp-db --file=schema.sql
```

### Functions에서 사용

```javascript
export async function onRequestGet(context) {
    const { env } = context;

    // SELECT
    const { results } = await env.DB.prepare(
        'SELECT * FROM posts ORDER BY created_at DESC LIMIT 10'
    ).all();

    // INSERT
    const { success, meta } = await env.DB.prepare(
        'INSERT INTO posts (user_id, title, content, created_at) VALUES (?, ?, ?, ?)'
    ).bind(1, 'Title', 'Content', new Date().toISOString()).run();

    // UPDATE
    await env.DB.prepare(
        'UPDATE posts SET title = ? WHERE id = ?'
    ).bind('New Title', 123).run();

    // DELETE
    await env.DB.prepare(
        'DELETE FROM posts WHERE id = ?'
    ).bind(123).run();

    return new Response(JSON.stringify({ results }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## KV 네임스페이스 바인딩

### KV 생성

```bash
# KV 네임스페이스 생성
npx wrangler kv:namespace create CACHE

# Preview 네임스페이스
npx wrangler kv:namespace create CACHE --preview
```

### wrangler.toml에 추가

```toml
[[kv_namespaces]]
binding = "CACHE"
id = "xxx-xxx-xxx"
preview_id = "yyy-yyy-yyy"
```

### Functions에서 사용

```javascript
export async function onRequestGet(context) {
    const { env, params } = context;
    const postId = params.id;

    // 캐시 확인
    const cached = await env.CACHE.get(`post:${postId}`);
    if (cached) {
        return new Response(cached, {
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // DB 조회
    const { results } = await env.DB.prepare(
        'SELECT * FROM posts WHERE id = ?'
    ).bind(postId).all();

    const post = results[0];

    // 캐시 저장 (1시간)
    await env.CACHE.put(`post:${postId}`, JSON.stringify(post), {
        expirationTtl: 3600
    });

    return new Response(JSON.stringify(post), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## R2 버킷 바인딩

### R2 생성

```bash
# R2 버킷 생성
npx wrangler r2 bucket create myapp-uploads
```

### wrangler.toml에 추가

```toml
[[r2_buckets]]
binding = "UPLOADS"
bucket_name = "myapp-uploads"
```

### Functions에서 사용

`functions/api/upload.js`:

```javascript
export async function onRequestPost(context) {
    const { request, env } = context;

    const formData = await request.formData();
    const file = formData.get('file');

    if (!file) {
        return new Response(JSON.stringify({
            success: false,
            message: '파일을 선택하세요.'
        }), {
            status: 400,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // 파일명 생성
    const filename = `${Date.now()}-${file.name}`;

    // R2에 업로드
    await env.UPLOADS.put(filename, file.stream(), {
        httpMetadata: {
            contentType: file.type
        }
    });

    // URL 생성
    const url = `https://r2.myapp.com/${filename}`;

    return new Response(JSON.stringify({
        success: true,
        url
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## 시크릿 관리

### 시크릿 설정

```bash
# 시크릿 추가
npx wrangler pages secret put API_KEY
# 프롬프트에서 값 입력

# 시크릿 목록
npx wrangler pages secret list

# 시크릿 삭제
npx wrangler pages secret delete API_KEY
```

### Functions에서 사용

```javascript
export async function onRequestGet(context) {
    const { env } = context;

    // 시크릿 사용
    const apiKey = env.API_KEY;
    const jwtSecret = env.JWT_SECRET;

    // 외부 API 호출
    const response = await fetch('https://api.example.com/data', {
        headers: {
            'Authorization': `Bearer ${apiKey}`
        }
    });

    // ...
}
```

## 로컬 개발 환경

### .dev.vars 파일

`.dev.vars` (로컬 개발용, .gitignore에 추가):

```
API_KEY=local_dev_key
DATABASE_URL=http://localhost:3306
DEBUG=true
JWT_SECRET=local_secret
```

### 로컬 서버 실행

```bash
# Pages 로컬 개발
npx wrangler pages dev .

# 특정 포트
npx wrangler pages dev . --port 8080

# D1 로컬 바인딩
npx wrangler pages dev . --d1 DB=myapp-db --local

# KV 로컬 바인딩
npx wrangler pages dev . --kv CACHE
```

## Claude Code와 함께 사용하기

### 환경 변수 설정 요청

```
Cloudflare Pages 환경 변수를 설정해줘.

Variables:
- API_KEY (Production): secret_xxx
- API_KEY (Preview): dev_xxx
- DATABASE_URL: https://db.example.com
- JWT_SECRET: (시크릿으로 설정)

wrangler.toml에 D1, KV 바인딩도 추가.
```

### D1 스키마 생성

```
D1 데이터베이스 스키마를 만들어줘.

테이블:
1. users (id, email, name, created_at)
2. posts (id, user_id, title, content, created_at)
3. comments (id, post_id, user_id, content, created_at)

외래 키 제약 조건 추가.
schema.sql 파일 생성 후 적용.
```

### 로컬 개발 설정

```
로컬 개발 환경을 설정해줘.

.dev.vars 파일 생성 (API_KEY, DATABASE_URL 등).
wrangler.toml에 로컬 바인딩 추가.
npm run dev 스크립트 추가 (wrangler pages dev).
```

## 체크리스트

환경 설정 확인사항:

- [ ] 환경 변수가 Cloudflare 대시보드에 설정되어 있는가?
- [ ] wrangler.toml에 바인딩이 설정되어 있는가?
- [ ] D1 스키마가 생성되어 있는가?
- [ ] KV/R2 네임스페이스가 생성되어 있는가?
- [ ] .dev.vars가 .gitignore에 추가되어 있는가?
- [ ] 로컬 개발 환경이 작동하는가?
- [ ] 프로덕션과 프리뷰 환경이 분리되어 있는가?

## 자주 하는 실수

### 1. .dev.vars 커밋

```bash
# ❌ 잘못된 설정 (.gitignore 없음)
git add .dev.vars
git commit -m "Add env vars"  # 시크릿 노출!

# ✅ 올바른 설정
echo ".dev.vars" >> .gitignore
git add .gitignore
git commit -m "Add gitignore"
```

### 2. 바인딩 이름 불일치

```toml
# wrangler.toml
[[d1_databases]]
binding = "DATABASE"  # DATABASE

# functions/api/posts.js
export async function onRequestGet(context) {
    const { env } = context;
    const { results } = await env.DB.prepare(...);  # ❌ DB (오류)
    const { results } = await env.DATABASE.prepare(...);  # ✅ DATABASE
}
```

### 3. 환경별 설정 누락

```toml
# ❌ 잘못된 설정 (프로덕션과 프리뷰가 같은 DB 사용)
[[d1_databases]]
binding = "DB"
database_id = "prod-xxx"

# ✅ 올바른 설정
[[d1_databases]]
binding = "DB"
database_id = "preview-xxx"  # 프리뷰 (기본)

[[env.production.d1_databases]]
binding = "DB"
database_id = "prod-xxx"  # 프로덕션
```

## 관련 문서

- [Functions 개발](pages-functions.md)
- [API 엔드포인트 개발](pages-api.md)
- [빌드 및 배포](pages-deployment.md)
- [Workers D1 데이터베이스 활용](workers-d1.md)
