# Cloudflare Workers - D1 데이터베이스 활용

Workers D1 데이터베이스를 활용한 데이터 관리 방법을 학습합니다.

## D1 기본

### D1 생성

```bash
# D1 데이터베이스 생성
npx wrangler d1 create mydb

# 출력에서 database_id 복사
```

### wrangler.toml 설정

```toml
[[d1_databases]]
binding = "DB"
database_name = "mydb"
database_id = "xxx-xxx-xxx"
```

## 스키마 관리

### schema.sql

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

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at);
```

### 스키마 적용

```bash
# 로컬
npx wrangler d1 execute mydb --local --file=schema.sql

# 프로덕션
npx wrangler d1 execute mydb --file=schema.sql
```

## CRUD 작업

### SELECT

```typescript
// 전체 조회
const { results } = await env.DB.prepare(
    'SELECT * FROM users'
).all();

// 단일 조회
const user = await env.DB.prepare(
    'SELECT * FROM users WHERE id = ?'
).bind(userId).first();

// 조건 조회
const { results } = await env.DB.prepare(
    'SELECT * FROM users WHERE email LIKE ?'
).bind(`%${keyword}%`).all();
```

### INSERT

```typescript
const { success, meta } = await env.DB.prepare(
    'INSERT INTO users (email, name, created_at) VALUES (?, ?, ?)'
).bind(email, name, new Date().toISOString()).run();

const newUserId = meta.last_row_id;
```

### UPDATE

```typescript
await env.DB.prepare(
    'UPDATE users SET name = ? WHERE id = ?'
).bind(newName, userId).run();
```

### DELETE

```typescript
await env.DB.prepare(
    'DELETE FROM users WHERE id = ?'
).bind(userId).run();
```

## 트랜잭션

```typescript
const results = await env.DB.batch([
    env.DB.prepare('INSERT INTO users (email, name) VALUES (?, ?)').bind('user1@example.com', 'User 1'),
    env.DB.prepare('INSERT INTO users (email, name) VALUES (?, ?)').bind('user2@example.com', 'User 2'),
    env.DB.prepare('INSERT INTO users (email, name) VALUES (?, ?)').bind('user3@example.com', 'User 3')
]);
```

## JOIN 쿼리

```typescript
const { results } = await env.DB.prepare(`
    SELECT p.*, u.name as author_name
    FROM posts p
    JOIN users u ON p.user_id = u.id
    ORDER BY p.created_at DESC
    LIMIT 10
`).all();
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [API 개발](workers-api.md)
