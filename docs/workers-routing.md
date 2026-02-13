# Cloudflare Workers - 라우팅

Hono 프레임워크 기반 라우팅, 미들웨어, 서비스 패턴을 안내합니다.

## 아키텍처

```
Request → Middleware → Route → Service → Response
```

- **Route**: HTTP 요청 수신, 서비스 호출 (비즈니스 로직 없음)
- **Service**: 비즈니스 로직 (클래스 기반, env 주입)
- **Middleware**: 인증, 에러 처리, 로깅
- **Utils**: 상태 없는 유틸리티 함수

## 앱 초기화 (src/index.js)

```javascript
import { Hono } from 'hono';
import { logger } from 'hono/logger';
import { cors } from 'hono/cors';
import { authMiddleware } from './middleware/auth.js';
import { errorHandler } from './middleware/errorHandler.js';
import authRoutes from './routes/auth.js';
import userRoutes from './routes/users.js';

const app = new Hono();

// 글로벌 미들웨어
app.use('*', logger());
app.use('*', cors());

// 인증 미들웨어 (PUBLIC_PATHS 제외)
const PUBLIC_PATHS = ['/health', '/docs', '/openapi.json', '/auth'];
app.use('*', async (c, next) => {
    const path = new URL(c.req.url).pathname;
    if (PUBLIC_PATHS.some(p => path.startsWith(p))) {
        return next();
    }
    return authMiddleware(c, next);
});

// 라우트 등록
app.route('/auth', authRoutes);
app.route('/users', userRoutes);

// 에러 핸들러
app.onError(errorHandler);

// 404
app.notFound((c) => c.json({ error: 'Not Found' }, 404));

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
    const userService = new UserService(c.env);
    const result = await userService.getUsers();
    return c.json(result);
});

// GET /users/profile (JWT에서 사용자 정보 추출)
users.get('/profile', async (c) => {
    const userId = c.get('userId');
    const userService = new UserService(c.env);
    const result = await userService.getUserById(userId);
    return c.json(result);
});

export default users;
```

### 라우트 규칙

- 핸들러는 얇게 유지 (비즈니스 로직은 Service에)
- `c.env`로 환경 변수 및 바인딩 접근
- `c.get('userId')`, `c.get('userRole')`로 인증 정보 접근
- `c.json()`으로 응답 반환

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
        // const { results } = await this.env.DB.prepare('SELECT * FROM users').all();
        // return formatResponse(results);

        // Mock 데이터
        return formatResponse([
            { id: 1, username: 'admin', role: 'admin' },
            { id: 2, username: 'user', role: 'user' }
        ]);
    }

    async getUserById(id) {
        // D1 사용 시:
        // const user = await this.env.DB.prepare('SELECT * FROM users WHERE id = ?').bind(id).first();
        // return formatResponse(user);

        return formatResponse({ id, username: 'user' });
    }
}
```

### 서비스 규칙

- 반드시 클래스로 작성
- 생성자에서 `env` 주입받기
- 에러는 `throw`로 전파 (errorHandler가 처리)

## 미들웨어

### JWT 인증 (src/middleware/auth.js)

```javascript
import * as jose from 'jose';

export async function authMiddleware(c, next) {
    const token = c.req.header('Authorization')?.replace('Bearer ', '');

    if (!token) {
        return c.json({ error: '인증이 필요합니다.' }, 401);
    }

    try {
        const secret = new TextEncoder().encode(c.env.JWT_SECRET);
        const { payload } = await jose.jwtVerify(token, secret);

        // 컨텍스트에 사용자 정보 저장
        c.set('userId', payload.sub || payload.userId);
        c.set('userEmail', payload.email);
        c.set('userRole', payload.role);

        await next();
    } catch (error) {
        return c.json({ error: '유효하지 않은 토큰입니다.' }, 401);
    }
}
```

### 에러 처리 (src/middleware/errorHandler.js)

```javascript
export function errorHandler(err, c) {
    console.error('Error:', err.message);

    const status = {
        'ValidationError': 400,
        'UnauthorizedError': 401
    }[err.name] || 500;

    return c.json({ error: err.message }, status);
}
```

### 에러 발생 방법

서비스에서 named error를 throw:

```javascript
const error = new Error('잘못된 입력입니다.');
error.name = 'ValidationError';  // → 400
throw error;
```

## 새 기능 추가 예제

### 1. 라우트 파일 생성

`src/routes/posts.js`:

```javascript
import { Hono } from 'hono';
import { PostService } from '../services/postService.js';

const posts = new Hono();

posts.get('/', async (c) => {
    const postService = new PostService(c.env);
    return c.json(await postService.getPosts());
});

posts.post('/', async (c) => {
    const data = await c.req.json();
    const userId = c.get('userId');
    const postService = new PostService(c.env);
    return c.json(await postService.createPost(data, userId), 201);
});

export default posts;
```

### 2. 서비스 파일 생성

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

### 3. index.js에 등록

```javascript
import postRoutes from './routes/posts.js';

app.route('/posts', postRoutes);
```

## 관련 문서

- [시작하기](workers-getting-started.md)
- [배포](workers-deployment.md)
