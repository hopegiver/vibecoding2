# Cloudflare Workers - KV 스토리지 활용

Workers KV를 활용한 캐싱과 데이터 저장 방법을 학습합니다.

## KV 기본

### KV 생성

```bash
# KV 네임스페이스 생성
npx wrangler kv:namespace create CACHE
npx wrangler kv:namespace create CACHE --preview
```

### wrangler.toml 설정

```toml
[[kv_namespaces]]
binding = "CACHE"
id = "xxx-xxx-xxx"
preview_id = "yyy-yyy-yyy"
```

## 기본 사용법

### 읽기/쓰기

```typescript
export default {
    async fetch(request: Request, env: Env): Promise<Response> {
        // 쓰기
        await env.CACHE.put('key', 'value');

        // 읽기
        const value = await env.CACHE.get('key');

        // 삭제
        await env.CACHE.delete('key');

        return Response.json({ value });
    }
};
```

### 만료 시간 설정

```typescript
// 60초 후 만료
await env.CACHE.put('key', 'value', {
    expirationTtl: 60
});

// 특정 시각에 만료
const expirationTime = Math.floor(Date.now() / 1000) + 3600; // 1시간 후
await env.CACHE.put('key', 'value', {
    expiration: expirationTime
});
```

## 캐싱 패턴

### API 응답 캐싱

```typescript
router.get('/api/posts/:id', async (request, env) => {
    const { id } = request.params;
    const cacheKey = `post:${id}`;

    // 캐시 확인
    const cached = await env.CACHE.get(cacheKey);
    if (cached) {
        return new Response(cached, {
            headers: { 'Content-Type': 'application/json', 'X-Cache': 'HIT' }
        });
    }

    // DB 조회
    const post = await env.DB.prepare(
        'SELECT * FROM posts WHERE id = ?'
    ).bind(id).first();

    // 캐시 저장 (1시간)
    await env.CACHE.put(cacheKey, JSON.stringify(post), {
        expirationTtl: 3600
    });

    return Response.json(post, {
        headers: { 'X-Cache': 'MISS' }
    });
});
```

### 캐시 무효화

```typescript
router.put('/api/posts/:id', async (request, env) => {
    const { id } = request.params;
    const data = await request.json();

    // DB 업데이트
    await env.DB.prepare(
        'UPDATE posts SET title = ?, content = ? WHERE id = ?'
    ).bind(data.title, data.content, id).run();

    // 캐시 무효화
    await env.CACHE.delete(`post:${id}`);

    return Response.json({ success: true });
});
```

## 리스트 작업

```typescript
// 모든 키 나열 (prefix 사용)
const { keys } = await env.CACHE.list({ prefix: 'post:' });

// 키 삭제
for (const key of keys) {
    await env.CACHE.delete(key.name);
}
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [D1 데이터베이스 활용](workers-d1.md)
