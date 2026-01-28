# Workers .claude 설정 예제

## 개요

이 문서는 Cloudflare Workers 프로젝트를 위한 `.claude` 폴더 설정 예제를 제공합니다. 실제 운영 중인 Workers 템플릿 프로젝트의 설정을 기반으로 작성되었습니다.

## 프로젝트 구조

```
your-workers-project/
├── .claude/
│   └── prompts.md                  # AI 개발자 가이드
├── CLAUDE.md                       # 프로젝트 컨텍스트 (선택)
├── CONTRIBUTING.md                 # 상세 개발 가이드
├── src/
│   ├── routes/                     # API 라우트
│   ├── services/                   # 비즈니스 로직 (클래스)
│   ├── middleware/                 # 미들웨어
│   └── index.js                    # 엔트리 포인트
└── wrangler.toml                   # Workers 설정
```

## 1. `.claude/prompts.md`

AI 개발자를 위한 핵심 가이드입니다.

```markdown
# Project Instructions

When starting work on this project, you MUST follow these steps:

## 1. Read Development Guide FIRST
Before writing any code, read [CONTRIBUTING.md](../CONTRIBUTING.md) in full.

This document contains:
- Coding conventions (file naming, class naming)
- Architecture patterns (service layer, route patterns)
- Cloudflare bindings usage (D1, KV, R2)
- 7 core rules for AI development

## 2. Follow the Standards
All code you write must follow the conventions defined in CONTRIBUTING.md:
- File names: camelCase (e.g., `authService.js`)
- Class names: PascalCase (e.g., `class AuthService`)
- Services: Class-based with `env` injection in constructor
- Routes: Use Hono router patterns
- No unnecessary abstractions

## 3. Check Examples
When adding new features, refer to existing code:
- Routes: `src/routes/auth.js`, `src/routes/users.js`
- Services: `src/services/authService.js`, `src/services/userService.js`
- Middleware: `src/middleware/auth.js`

## 4. Never Skip the Guide
Even if the user doesn't mention CONTRIBUTING.md, you must reference it when writing code.
Consistency with existing patterns is critical for this template.
```

## 2. `CONTRIBUTING.md`

프로젝트의 상세한 개발 가이드입니다. 이 파일은 `.claude/prompts.md`에서 참조됩니다.

```markdown
# 개발 가이드 (Development Guide)

이 문서는 Cloudflare Workers 프로젝트의 개발 표준 가이드입니다.

## 프로젝트 구조

\`\`\`
src/
├── routes/          # API 라우트 핸들러
├── services/        # 비즈니스 로직 (클래스 기반)
├── utils/           # 유틸리티 함수
├── middleware/      # 미들웨어
└── index.js         # 엔트리 포인트
\`\`\`

### 폴더별 역할

#### `routes/`
- HTTP 요청/응답 처리
- 입력 검증
- 서비스 호출
- **규칙**: 비즈니스 로직 포함 금지, 서비스 레이어에 위임

#### `services/`
- 비즈니스 로직
- 외부 API 통합
- 데이터 처리
- **규칙**: 반드시 클래스 기반, `constructor(env)` 패턴

#### `middleware/`
- 인증, 로깅, CORS 등
- 라우트 전후 처리
- **규칙**: Hono 미들웨어 패턴 사용

## 코딩 컨벤션

### 파일명
- **서비스**: `camelCase` (예: `authService.js`, `userService.js`)
- **라우트**: `camelCase` (예: `users.js`, `auth.js`)
- **유틸리티**: `camelCase` (예: `utils.js`)

### 변수/함수명
- **변수**: `camelCase`
- **함수**: `camelCase`
- **클래스**: `PascalCase`
- **상수**: `UPPER_SNAKE_CASE`

### Export 패턴

\`\`\`javascript
// ✅ 좋은 예: Services (클래스)
export class UserService {
  constructor(env) { ... }
}

// ✅ 좋은 예: Utils (객체)
export const KV = {
  async get(kv, key) { ... }
}

// ❌ 나쁜 예: default export 남용
export default function() { ... }
\`\`\`

## 아키텍처 패턴

### 레이어 구조

\`\`\`
Request → Route → Service → Utils/Bindings → Response
\`\`\`

### 서비스 레이어 패턴

모든 서비스는 **클래스 기반**으로 작성합니다.

\`\`\`javascript
// ✅ 올바른 서비스 패턴
export class UserService {
  constructor(env) {
    this.env = env;
  }

  async getUser(userId) {
    // KV 캐시 확인
    const cached = await this.env.KV.get(\`user:\${userId}\`, { type: 'json' });
    if (cached) return cached;

    // D1 조회
    const user = await this.env.DB
      .prepare('SELECT * FROM users WHERE id = ?')
      .bind(userId)
      .first();

    // 캐시 저장
    if (user) {
      await this.env.KV.put(\`user:\${userId}\`, JSON.stringify(user), {
        expirationTtl: 3600
      });
    }

    return user;
  }
}
\`\`\`

### 라우트 패턴

\`\`\`javascript
// ✅ 올바른 라우트 패턴
import { Hono } from 'hono';
import { UserService } from '../services/userService.js';

const users = new Hono();

users.get('/:id', async (c) => {
  const userId = c.req.param('id');

  // 서비스 인스턴스 생성
  const userService = new UserService(c.env);

  // 비즈니스 로직은 서비스에 위임
  const user = await userService.getUser(userId);

  return c.json({ data: user });
});

export default users;
\`\`\`

## Cloudflare 바인딩 사용

Cloudflare 바인딩은 **직접 사용**합니다.

### KV (Key-Value Storage)
\`\`\`javascript
await c.env.KV.put('key', 'value', { expirationTtl: 3600 });
const value = await c.env.KV.get('key', { type: 'json' });
await c.env.KV.delete('key');
\`\`\`

### D1 (SQLite Database)
\`\`\`javascript
const user = await c.env.DB
  .prepare('SELECT * FROM users WHERE id = ?')
  .bind(userId)
  .first();
\`\`\`

### R2 (Object Storage)
\`\`\`javascript
await c.env.BUCKET.put('file.txt', 'Hello World');
const object = await c.env.BUCKET.get('file.txt');
await c.env.BUCKET.delete('file.txt');
\`\`\`

## 인증 및 보안

### 공개 경로 설정
\`\`\`javascript
// src/index.js
const PUBLIC_PATHS = ['/health', '/docs', '/auth'];
\`\`\`

### 인증된 사용자 정보 접근
\`\`\`javascript
users.get('/profile', async (c) => {
  const userId = c.get('userId');
  const userEmail = c.get('userEmail');
  const userRole = c.get('userRole');
  // ...
});
\`\`\`

## AI 개발 시 참고사항

이 프로젝트를 AI와 함께 개발할 때 다음을 명심하세요:

1. **서비스는 항상 클래스로 작성**
2. **utils는 stateless 함수로 작성**
3. **라우트에는 비즈니스 로직 금지**
4. **파일명은 camelCase**
5. **env는 생성자에서 주입**
6. **Cloudflare 바인딩은 직접 사용** (c.env.KV, c.env.DB, c.env.BUCKET)
7. **인증이 필요한 라우트는 PUBLIC_PATHS에서 제외**
```

## 3. `CLAUDE.md` (선택)

간단한 프로젝트 개요를 제공합니다. CONTRIBUTING.md와 중복되는 내용은 최소화합니다.

```markdown
# Cloudflare Workers API 템플릿

## 프로젝트 개요
Cloudflare Workers 기반 REST API 서버 템플릿. JWT 인증, D1 데이터베이스, KV 캐시를 포함한 표준 백엔드 구조.

**기술 스택:** Hono + JWT + Cloudflare Workers (D1, KV, R2)

## 빠른 참조

### 프로젝트 구조
\`\`\`
src/
├── routes/          # API 라우트
├── services/        # 비즈니스 로직 (클래스)
├── middleware/      # 미들웨어
└── index.js         # 엔트리 포인트
\`\`\`

### 아키텍처
\`\`\`
Request → Route → Service → Bindings → Response
\`\`\`

### 필수 규칙
1. ✅ 서비스는 클래스로, utils는 함수로
2. ✅ 라우트에는 비즈니스 로직 금지
3. ✅ 파일명은 camelCase
4. ✅ env는 생성자에서 주입
5. ✅ Cloudflare 바인딩 직접 사용

---
**상세 가이드:** [CONTRIBUTING.md](CONTRIBUTING.md) 참조
```

## 4. 실전 활용 예시

### 새로운 API 엔드포인트 추가 요청

```
프롬프트: "상품(products) CRUD API를 만들어줘. D1 데이터베이스를 사용하고,
KV로 캐시도 추가해줘."
```

**Claude Code의 동작:**
1. `.claude/prompts.md` 읽고 `CONTRIBUTING.md` 확인
2. `src/services/productService.js` 생성 (클래스 기반)
   - 생성자에서 `env` 주입
   - D1 조회 로직
   - KV 캐시 로직
3. `src/routes/products.js` 생성
   - 서비스만 호출
   - 비즈니스 로직 없음
4. `src/index.js`에 라우트 등록

**생성된 코드 예시:**

`src/services/productService.js`:
```javascript
export class ProductService {
  constructor(env) {
    this.env = env;
  }

  async getProduct(productId) {
    // KV 캐시 확인
    const cacheKey = `product:${productId}`;
    const cached = await this.env.KV.get(cacheKey, { type: 'json' });
    if (cached) return cached;

    // D1 조회
    const product = await this.env.DB
      .prepare('SELECT * FROM products WHERE id = ?')
      .bind(productId)
      .first();

    // 캐시 저장 (1시간)
    if (product) {
      await this.env.KV.put(cacheKey, JSON.stringify(product), {
        expirationTtl: 3600
      });
    }

    return product;
  }

  async createProduct(data) {
    const result = await this.env.DB
      .prepare('INSERT INTO products (name, price) VALUES (?, ?)')
      .bind(data.name, data.price)
      .run();

    return result.meta.last_row_id;
  }
}
```

`src/routes/products.js`:
```javascript
import { Hono } from 'hono';
import { ProductService } from '../services/productService.js';

const products = new Hono();

products.get('/:id', async (c) => {
  const productId = c.req.param('id');
  const productService = new ProductService(c.env);
  const product = await productService.getProduct(productId);

  if (!product) {
    return c.json({ error: 'Product not found' }, 404);
  }

  return c.json({ data: product });
});

products.post('/', async (c) => {
  const body = await c.req.json();
  const productService = new ProductService(c.env);
  const productId = await productService.createProduct(body);

  return c.json({ data: { id: productId } }, 201);
});

export default products;
```

### 외부 API 연동 요청

```
프롬프트: "OpenAI API를 호출하는 서비스를 만들어줘."
```

**Claude Code의 동작:**
1. `CONTRIBUTING.md`의 서비스 패턴 확인
2. `src/services/openaiService.js` 생성
3. 환경 변수 `OPENAI_API_KEY` 사용
4. 클래스 기반으로 작성

**생성된 코드:**
```javascript
export class OpenAIService {
  constructor(env) {
    this.apiKey = env.OPENAI_API_KEY;
    this.baseURL = 'https://api.openai.com/v1';
  }

  async chat(messages) {
    const response = await fetch(`${this.baseURL}/chat/completions`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`
      },
      body: JSON.stringify({
        model: 'gpt-4',
        messages
      })
    });

    return await response.json();
  }
}
```

## 효과

이 `.claude` 설정으로:
- ✅ 서비스 레이어를 일관되게 클래스로 작성
- ✅ 라우트에 비즈니스 로직이 들어가는 실수 방지
- ✅ Cloudflare 바인딩을 올바르게 사용
- ✅ 파일명과 클래스명 컨벤션 자동 준수
- ✅ 인증 패턴을 자동으로 적용

## 비교: `.claude` 설정 전후

### 설정 전

**프롬프트:** "상품 API 만들어줘"

**Claude Code 결과:**
```javascript
// ❌ 라우트에 모든 로직 포함
products.get('/:id', async (c) => {
  // DB 직접 조회
  const product = await c.env.DB
    .prepare('SELECT * FROM products WHERE id = ?')
    .bind(c.req.param('id'))
    .first();

  return c.json(product);
});
```

### 설정 후

**프롬프트:** "상품 API 만들어줘"

**Claude Code 결과:**
```javascript
// ✅ 서비스 레이어 분리
export class ProductService {
  constructor(env) {
    this.env = env;
  }
  // 비즈니스 로직
}

// ✅ 라우트는 서비스만 호출
products.get('/:id', async (c) => {
  const service = new ProductService(c.env);
  const product = await service.getProduct(c.req.param('id'));
  return c.json({ data: product });
});
```

## 다음 단계

- [Pages .claude 설정 예제](pages-claude-setup.md)
- [맑은프레임워크 .claude 설정 예제](malgn-claude-setup.md)
- [MCP 서버 설정 및 활용](mcp-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
