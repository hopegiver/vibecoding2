# Cloudflare Workers - 라우팅 및 요청 처리

Workers에서 HTTP 요청을 라우팅하고 처리하는 방법을 학습합니다.

## 기본 라우팅

### URL 파싱

```typescript
export default {
    async fetch(request: Request): Promise<Response> {
        const url = new URL(request.url);
        const path = url.pathname;
        const method = request.method;

        // 쿼리 파라미터
        const name = url.searchParams.get('name');

        if (path === '/' && method === 'GET') {
            return new Response('Home');
        }

        if (path === '/api/hello' && method === 'GET') {
            return new Response(`Hello, ${name || 'World'}!`);
        }

        return new Response('Not Found', { status: 404 });
    }
};
```

## 고급 라우터 구현

### itty-router 사용

```bash
npm install itty-router
```

```typescript
import { Router } from 'itty-router';

const router = Router();

router.get('/', () => {
    return new Response('API is running');
});

router.get('/api/users', async (request, env) => {
    const users = await env.DB.prepare('SELECT * FROM users').all();
    return Response.json(users.results);
});

router.get('/api/users/:id', async (request, env) => {
    const { id } = request.params;
    const user = await env.DB.prepare('SELECT * FROM users WHERE id = ?').bind(id).first();
    return Response.json(user);
});

router.post('/api/users', async (request, env) => {
    const data = await request.json();
    await env.DB.prepare('INSERT INTO users (name, email) VALUES (?, ?)').bind(data.name, data.email).run();
    return Response.json({ success: true }, { status: 201 });
});

export default {
    fetch: router.handle
};
```

## 요청 처리

### JSON 요청

```typescript
router.post('/api/posts', async (request) => {
    try {
        const data = await request.json();

        // 유효성 검사
        if (!data.title || !data.content) {
            return Response.json({
                error: '제목과 내용을 입력하세요.'
            }, { status: 400 });
        }

        // 처리...
        return Response.json({ success: true });
    } catch (error) {
        return Response.json({
            error: '잘못된 요청입니다.'
        }, { status: 400 });
    }
});
```

### FormData 처리

```typescript
router.post('/api/upload', async (request) => {
    const formData = await request.formData();
    const file = formData.get('file') as File;
    const title = formData.get('title') as string;

    if (!file) {
        return Response.json({ error: '파일을 선택하세요.' }, { status: 400 });
    }

    // 파일 처리...
    return Response.json({ success: true });
});
```

## 미들웨어

### CORS 미들웨어

```typescript
function corsHeaders() {
    return {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization'
    };
}

router.options('*', () => {
    return new Response(null, {
        headers: corsHeaders()
    });
});

router.all('*', async (request, env) => {
    const response = await router.handle(request, env);
    Object.entries(corsHeaders()).forEach(([key, value]) => {
        response.headers.set(key, value);
    });
    return response;
});
```

### 인증 미들웨어

```typescript
async function authMiddleware(request: Request, env: Env) {
    const token = request.headers.get('Authorization')?.replace('Bearer ', '');

    if (!token) {
        return Response.json({ error: '인증이 필요합니다.' }, { status: 401 });
    }

    // JWT 검증
    const isValid = await verifyToken(token, env.JWT_SECRET);
    if (!isValid) {
        return Response.json({ error: '유효하지 않은 토큰입니다.' }, { status: 403 });
    }

    return null; // 통과
}

router.get('/api/protected', async (request, env) => {
    const authError = await authMiddleware(request, env);
    if (authError) return authError;

    // 보호된 리소스
    return Response.json({ data: 'Protected data' });
});
```

## 체크리스트

- [ ] 라우터가 모든 경로를 처리하는가?
- [ ] 404 처리가 되어 있는가?
- [ ] CORS가 설정되어 있는가?
- [ ] 파라미터 유효성 검사를 했는가?
- [ ] 오류 처리가 적절한가?

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [API 개발](workers-api.md)
