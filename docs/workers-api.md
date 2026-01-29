# Cloudflare Workers - API 개발

RESTful API 개발 패턴과 베스트 프랙티스를 학습합니다.

## RESTful API 패턴

### CRUD 엔드포인트

```typescript
import { Router } from 'itty-router';

const router = Router();

// GET /api/posts - 목록
router.get('/api/posts', async (request, env) => {
    const { results } = await env.DB.prepare(
        'SELECT * FROM posts ORDER BY created_at DESC'
    ).all();

    return Response.json(results);
});

// GET /api/posts/:id - 상세
router.get('/api/posts/:id', async (request, env) => {
    const { id } = request.params;
    const post = await env.DB.prepare(
        'SELECT * FROM posts WHERE id = ?'
    ).bind(id).first();

    if (!post) {
        return Response.json({ error: 'Not found' }, { status: 404 });
    }

    return Response.json(post);
});

// POST /api/posts - 생성
router.post('/api/posts', async (request, env) => {
    const data = await request.json();

    const { success, meta } = await env.DB.prepare(
        'INSERT INTO posts (title, content) VALUES (?, ?)'
    ).bind(data.title, data.content).run();

    return Response.json({ id: meta.last_row_id }, { status: 201 });
});

// PUT /api/posts/:id - 수정
router.put('/api/posts/:id', async (request, env) => {
    const { id } = request.params;
    const data = await request.json();

    await env.DB.prepare(
        'UPDATE posts SET title = ?, content = ? WHERE id = ?'
    ).bind(data.title, data.content, id).run();

    return Response.json({ success: true });
});

// DELETE /api/posts/:id - 삭제
router.delete('/api/posts/:id', async (request, env) => {
    const { id } = request.params;

    await env.DB.prepare('DELETE FROM posts WHERE id = ?').bind(id).run();

    return Response.json({ success: true });
});

export default { fetch: router.handle };
```

## 에러 처리

```typescript
class ApiError extends Error {
    constructor(public statusCode: number, message: string) {
        super(message);
    }
}

async function handleRequest(request: Request, env: Env) {
    try {
        // 요청 처리
        return Response.json({ success: true });
    } catch (error) {
        if (error instanceof ApiError) {
            return Response.json({
                error: error.message
            }, { status: error.statusCode });
        }

        return Response.json({
            error: 'Internal Server Error'
        }, { status: 500 });
    }
}
```

## 페이징

```typescript
router.get('/api/posts', async (request, env) => {
    const url = new URL(request.url);
    const page = parseInt(url.searchParams.get('page') || '1');
    const limit = parseInt(url.searchParams.get('limit') || '10');
    const offset = (page - 1) * limit;

    const { results: posts } = await env.DB.prepare(
        'SELECT * FROM posts ORDER BY created_at DESC LIMIT ? OFFSET ?'
    ).bind(limit, offset).all();

    const { total } = await env.DB.prepare(
        'SELECT COUNT(*) as total FROM posts'
    ).first() as any;

    return Response.json({
        data: posts,
        pagination: {
            page,
            limit,
            total,
            totalPages: Math.ceil(total / limit)
        }
    });
});
```

## 관련 문서

- [라우팅 및 요청 처리](workers-routing.md)
- [D1 데이터베이스 활용](workers-d1.md)
