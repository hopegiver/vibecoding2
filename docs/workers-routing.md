# Cloudflare Workers - 라우팅

Hono 프레임워크 기반 라우팅, 미들웨어, 서비스 패턴을 안내합니다.

## 아키텍처

```
Request → Middleware → Route → Service → Response
```

- **Route**: HTTP 요청 수신, 입력 검증, 서비스 호출 (비즈니스 로직 없음)
- **Service**: 비즈니스 로직 (클래스 기반, env 주입)
- **Middleware**: 인증, 에러 처리, 로깅, CORS
- **Utils**: 상태 없는 유틸리티 함수

## 앱 초기화 (src/index.js)

```javascript
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { logger } from 'hono/logger';
import { swaggerUI } from '@hono/swagger-ui';
import openApiSpec from './openapi.js';
import { errorHandler } from './middleware/errorHandler.js';
import { authMiddleware } from './middleware/auth.js';
import authRoutes from './routes/auth.js';
import usersRoutes from './routes/users.js';

const app = new Hono();

// 인증 불필요 경로
const PUBLIC_PATHS = ['/health', '/docs', '/openapi.json', '/auth'];

// 글로벌 미들웨어
app.use('*', logger());
app.use('*', cors());

// 인증 미들웨어 (PUBLIC_PATHS 제외)
app.use('*', async (c, next) => {
  const path = c.req.path;
  if (PUBLIC_PATHS.some(p => path.startsWith(p))) {
    return await next();
  }
  return await authMiddleware(c, next);
});

// 라우트 등록
app.route('/auth', authRoutes);
app.route('/users', usersRoutes);

// API 문서
app.get('/docs', swaggerUI({ url: '/openapi.json' }));
app.get('/openapi.json', (c) => c.json(openApiSpec));

// Health check
app.get('/health', (c) => c.json({ status: 'healthy', timestamp: new Date().toISOString() }));

// 에러 핸들러
app.onError(errorHandler);

// 404
app.notFound((c) => c.json({ error: 'Not Found', path: c.req.path }, 404));

export default app;
```

## 라우트 작성

### 기본 패턴

`src/routes/users.js`:

```javascript
import { Hono } from 'hono';
import { UserService } from '../services/userService.js';

const users = new Hono();

// GET /users
users.get('/', async (c) => {
  try {
    const userService = new UserService(c.env);
    const users = await userService.getUsers();
    return c.json({ data: users });
  } catch (error) {
    throw error;
  }
});

// GET /users/profile (JWT에서 사용자 정보 추출)
users.get('/profile', async (c) => {
  const userId = c.get('userId');
  const userEmail = c.get('userEmail');
  const userRole = c.get('userRole');

  const userService = new UserService(c.env);
  const user = await userService.getUserById(userId);

  return c.json({
    data: {
      ...user,
      email: userEmail,
      role: userRole
    }
  });
});

export default users;
```

### 라우트 규칙

- 핸들러는 얇게 유지 (입력 검증 + 서비스 호출만)
- `c.env`로 환경 변수 및 바인딩 접근
- `c.get('userId')`, `c.get('userEmail')`, `c.get('userRole')`로 인증 정보 접근
- `c.json()`으로 응답 반환
- 에러는 `throw`로 전파 (errorHandler가 처리)

## 서비스 작성

### 기본 패턴

`src/services/userService.js`:

```javascript
import { formatResponse } from '../utils/utils.js';

export class UserService {
  constructor(env) {
    this.env = env;
  }

  async getUsers() {
    // D1 사용 시:
    // const { results } = await this.env.DB
    //   .prepare('SELECT * FROM users')
    //   .all();
    // return formatResponse(results);

    // Mock 데이터
    const mockUsers = [
      { id: 1, name: 'John Doe', email: 'john@example.com' },
      { id: 2, name: 'Jane Smith', email: 'jane@example.com' }
    ];
    return formatResponse(mockUsers);
  }

  async getUserById(id) {
    // D1 + KV 캐시 사용 시:
    // const cached = await this.env.KV.get(`user:${id}`, { type: 'json' });
    // if (cached) return cached;
    //
    // const user = await this.env.DB
    //   .prepare('SELECT * FROM users WHERE id = ?')
    //   .bind(id)
    //   .first();
    //
    // if (user) {
    //   await this.env.KV.put(`user:${id}`, JSON.stringify(user), {
    //     expirationTtl: 3600
    //   });
    // }
    // return formatResponse(user);

    const mockUser = { id, name: 'John Doe', email: 'john@example.com' };
    return formatResponse(mockUser);
  }
}
```

### 서비스 규칙

- 반드시 클래스로 작성
- 생성자에서 `env` 주입받기
- `formatResponse()`로 응답 데이터 포맷
- 에러는 named error로 `throw` (errorHandler가 처리)

## 미들웨어

### JWT 인증 (src/middleware/auth.js)

```javascript
import { jwtVerify } from 'jose';

export const authMiddleware = async (c, next) => {
  const authHeader = c.req.header('Authorization');

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    const error = new Error('Missing or invalid authorization header');
    error.name = 'UnauthorizedError';
    throw error;
  }

  const token = authHeader.substring(7);

  try {
    const secret = c.env.JWT_SECRET;
    const encoder = new TextEncoder();
    const { payload } = await jwtVerify(token, encoder.encode(secret), {
      algorithms: ['HS256']
    });

    // 컨텍스트에 사용자 정보 저장
    c.set('userId', payload.sub || payload.userId);
    c.set('userEmail', payload.email);
    c.set('userRole', payload.role);
    c.set('jwtPayload', payload);

    await next();
  } catch (err) {
    const error = new Error('Invalid or expired token');
    error.name = 'UnauthorizedError';
    throw error;
  }
};
```

### 에러 처리 (src/middleware/errorHandler.js)

```javascript
export const errorHandler = (err, c) => {
  console.error('Error:', err);

  if (err.name === 'ValidationError') {
    return c.json({ error: 'Validation Error', message: err.message }, 400);
  }

  if (err.name === 'UnauthorizedError') {
    return c.json({ error: 'Unauthorized', message: err.message }, 401);
  }

  return c.json({
    error: 'Internal Server Error',
    message: err.message || 'Something went wrong'
  }, 500);
};
```

### 에러 발생 방법

서비스에서 named error를 throw:

```javascript
const error = new Error('잘못된 입력입니다.');
error.name = 'ValidationError';  // → 400
throw error;
```

## 새 기능 추가 예제

### 1. 서비스 클래스 생성

`src/services/postService.js`:

```javascript
import { formatResponse } from '../utils/utils.js';

export class PostService {
  constructor(env) {
    this.env = env;
  }

  async getPosts() {
    const { results } = await this.env.DB
      .prepare('SELECT * FROM posts ORDER BY created_at DESC')
      .all();
    return formatResponse(results);
  }

  async createPost(data, userId) {
    if (!data.title || !data.content) {
      const error = new Error('제목과 내용은 필수입니다.');
      error.name = 'ValidationError';
      throw error;
    }

    await this.env.DB
      .prepare('INSERT INTO posts (title, content, user_id) VALUES (?, ?, ?)')
      .bind(data.title, data.content, userId)
      .run();

    return formatResponse({ success: true });
  }
}
```

### 2. 라우트 핸들러 생성

`src/routes/posts.js`:

```javascript
import { Hono } from 'hono';
import { PostService } from '../services/postService.js';

const posts = new Hono();

posts.get('/', async (c) => {
  const postService = new PostService(c.env);
  return c.json({ data: await postService.getPosts() });
});

posts.post('/', async (c) => {
  const data = await c.req.json();
  const userId = c.get('userId');
  const postService = new PostService(c.env);
  return c.json({ data: await postService.createPost(data, userId) }, 201);
});

export default posts;
```

### 3. index.js에 등록

```javascript
import postRoutes from './routes/posts.js';

app.route('/posts', postRoutes);
```

### 4. openapi.js에 스펙 추가

## 관련 문서

- [시작하기](workers-getting-started.md)
- [배포](workers-deployment.md)
