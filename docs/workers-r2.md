# Cloudflare Workers - R2 객체 스토리지 활용

Workers R2를 활용한 파일 저장 및 관리 방법을 학습합니다.

## R2 기본

### R2 버킷 생성

```bash
npx wrangler r2 bucket create my-uploads
```

### wrangler.toml 설정

```toml
[[r2_buckets]]
binding = "UPLOADS"
bucket_name = "my-uploads"
```

## 파일 업로드

```typescript
router.post('/api/upload', async (request, env) => {
    const formData = await request.formData();
    const file = formData.get('file') as File;

    if (!file) {
        return Response.json({ error: '파일을 선택하세요.' }, { status: 400 });
    }

    // 파일명 생성
    const filename = `${Date.now()}-${file.name}`;

    // R2에 업로드
    await env.UPLOADS.put(filename, file.stream(), {
        httpMetadata: {
            contentType: file.type
        }
    });

    return Response.json({
        success: true,
        filename,
        url: `https://r2.example.com/${filename}`
    });
});
```

## 파일 다운로드

```typescript
router.get('/files/:filename', async (request, env) => {
    const { filename } = request.params;

    const object = await env.UPLOADS.get(filename);

    if (!object) {
        return new Response('Not Found', { status: 404 });
    }

    return new Response(object.body, {
        headers: {
            'Content-Type': object.httpMetadata.contentType || 'application/octet-stream'
        }
    });
});
```

## 파일 삭제

```typescript
router.delete('/api/files/:filename', async (request, env) => {
    const { filename } = request.params;

    await env.UPLOADS.delete(filename);

    return Response.json({ success: true });
});
```

## 관련 문서

- [프로젝트 시작하기](workers-getting-started.md)
- [API 개발](workers-api.md)
