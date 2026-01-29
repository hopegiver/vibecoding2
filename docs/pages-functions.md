# Cloudflare Pages - Functions 개발

Cloudflare Pages Functions를 활용한 서버리스 백엔드 API 개발 방법을 학습합니다.

## Functions 기본

### Functions란?

Cloudflare Pages Functions는 `/functions` 디렉토리의 파일을 자동으로 API 엔드포인트로 변환합니다.

**특징:**
- 서버 없이 백엔드 구현
- 파일 경로가 곧 API 경로
- HTTP 메서드별 처리 가능
- Edge에서 실행 (빠른 응답)

### 기본 구조

```
myapp/
├── functions/
│   ├── api/
│   │   ├── contact.js           # /api/contact
│   │   ├── newsletter.js        # /api/newsletter
│   │   └── posts/
│   │       ├── [id].js          # /api/posts/123
│   │       └── index.js         # /api/posts
│   └── _middleware.js           # 미들웨어
```

## 기본 Function

### GET 요청 처리

`functions/api/hello.js`:

```javascript
export async function onRequestGet(context) {
    return new Response(JSON.stringify({
        message: 'Hello from Cloudflare Pages!'
    }), {
        headers: {
            'Content-Type': 'application/json'
        }
    });
}
```

**URL:** `https://myapp.pages.dev/api/hello`

### POST 요청 처리

`functions/api/contact.js`:

```javascript
export async function onRequestPost(context) {
    const { request } = context;

    try {
        // JSON 데이터 파싱
        const data = await request.json();

        // 유효성 검사
        if (!data.name || !data.email || !data.message) {
            return new Response(JSON.stringify({
                success: false,
                message: '모든 필드를 입력하세요.'
            }), {
                status: 400,
                headers: { 'Content-Type': 'application/json' }
            });
        }

        // 이메일 전송 또는 DB 저장
        // ...

        return new Response(JSON.stringify({
            success: true,
            message: '메시지가 전송되었습니다.'
        }), {
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            success: false,
            message: '오류가 발생했습니다.'
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

## HTTP 메서드별 처리

### 모든 메서드 지원

`functions/api/posts/index.js`:

```javascript
// GET /api/posts - 목록 조회
export async function onRequestGet(context) {
    const posts = await fetchPostsFromDB();

    return new Response(JSON.stringify(posts), {
        headers: { 'Content-Type': 'application/json' }
    });
}

// POST /api/posts - 새 글 작성
export async function onRequestPost(context) {
    const { request } = context;
    const data = await request.json();

    const newPost = await createPost(data);

    return new Response(JSON.stringify({
        success: true,
        id: newPost.id
    }), {
        status: 201,
        headers: { 'Content-Type': 'application/json' }
    });
}

// PUT /api/posts - 글 수정
export async function onRequestPut(context) {
    const { request } = context;
    const data = await request.json();

    await updatePost(data.id, data);

    return new Response(JSON.stringify({
        success: true
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}

// DELETE /api/posts - 글 삭제
export async function onRequestDelete(context) {
    const { request } = context;
    const { searchParams } = new URL(request.url);
    const id = searchParams.get('id');

    await deletePost(id);

    return new Response(JSON.stringify({
        success: true
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## 동적 경로

### 파라미터 처리

`functions/api/posts/[id].js`:

```javascript
export async function onRequestGet(context) {
    const { params } = context;
    const postId = params.id;

    try {
        const post = await fetchPostById(postId);

        if (!post) {
            return new Response(JSON.stringify({
                error: '게시글을 찾을 수 없습니다.'
            }), {
                status: 404,
                headers: { 'Content-Type': 'application/json' }
            });
        }

        return new Response(JSON.stringify(post), {
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            error: '서버 오류'
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

**URL:** `/api/posts/123` → `params.id = "123"`

### 여러 파라미터

`functions/api/users/[userId]/posts/[postId].js`:

```javascript
export async function onRequestGet(context) {
    const { params } = context;
    const { userId, postId } = params;

    const post = await fetchUserPost(userId, postId);

    return new Response(JSON.stringify(post), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

**URL:** `/api/users/42/posts/123` → `{ userId: "42", postId: "123" }`

## Context 객체

### 사용 가능한 속성

```javascript
export async function onRequestGet(context) {
    const {
        request,     // Request 객체
        env,         // 환경 변수
        params,      // URL 파라미터
        data,        // 미들웨어에서 전달된 데이터
        next,        // 다음 함수 호출
        waitUntil    // 비동기 작업
    } = context;

    // 예제
    const url = new URL(request.url);
    const method = request.method;
    const headers = request.headers;

    return new Response('OK');
}
```

### 쿼리 파라미터

```javascript
export async function onRequestGet(context) {
    const { request } = context;
    const { searchParams } = new URL(request.url);

    const page = searchParams.get('page') || '1';
    const limit = searchParams.get('limit') || '10';

    const posts = await fetchPosts({ page, limit });

    return new Response(JSON.stringify(posts), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

**URL:** `/api/posts?page=2&limit=20`

## 환경 변수

### .env 파일

```
DATABASE_URL=https://...
API_KEY=secret_key
```

### Functions에서 사용

```javascript
export async function onRequestGet(context) {
    const { env } = context;

    const db = await connectToDatabase(env.DATABASE_URL);
    const data = await db.query('SELECT * FROM posts');

    return new Response(JSON.stringify(data), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## 미들웨어

### CORS 설정

`functions/_middleware.js`:

```javascript
export async function onRequest(context) {
    const { request, next } = context;

    // CORS 헤더 추가
    const response = await next();

    response.headers.set('Access-Control-Allow-Origin', '*');
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type');

    return response;
}
```

### 인증 체크

`functions/api/_middleware.js`:

```javascript
export async function onRequest(context) {
    const { request, next, env } = context;

    // 인증 토큰 확인
    const token = request.headers.get('Authorization');

    if (!token) {
        return new Response(JSON.stringify({
            error: '인증이 필요합니다.'
        }), {
            status: 401,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // 토큰 검증
    const isValid = await verifyToken(token, env.JWT_SECRET);

    if (!isValid) {
        return new Response(JSON.stringify({
            error: '유효하지 않은 토큰입니다.'
        }), {
            status: 403,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // 다음 함수 호출
    return next();
}
```

## D1 데이터베이스 연동

### D1 바인딩

`wrangler.toml`:

```toml
name = "myapp"

[[d1_databases]]
binding = "DB"
database_name = "myapp-db"
database_id = "xxx-xxx-xxx"
```

### Functions에서 사용

`functions/api/posts/index.js`:

```javascript
export async function onRequestGet(context) {
    const { env } = context;

    const { results } = await env.DB.prepare(
        'SELECT * FROM posts ORDER BY created_at DESC LIMIT 10'
    ).all();

    return new Response(JSON.stringify(results), {
        headers: { 'Content-Type': 'application/json' }
    });
}

export async function onRequestPost(context) {
    const { request, env } = context;
    const data = await request.json();

    const { success } = await env.DB.prepare(
        'INSERT INTO posts (title, content, created_at) VALUES (?, ?, ?)'
    ).bind(data.title, data.content, new Date().toISOString()).run();

    return new Response(JSON.stringify({
        success
    }), {
        status: 201,
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## KV 스토리지 연동

### KV 바인딩

`wrangler.toml`:

```toml
[[kv_namespaces]]
binding = "CACHE"
id = "xxx-xxx-xxx"
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

    // DB에서 조회
    const post = await fetchPostFromDB(postId);

    // 캐시 저장 (1시간)
    await env.CACHE.put(`post:${postId}`, JSON.stringify(post), {
        expirationTtl: 3600
    });

    return new Response(JSON.stringify(post), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

## 실전 프롬프트 예시

### 뉴스레터 구독 API

```
뉴스레터 구독 API를 만들어줘.

경로: /api/newsletter
메서드: POST
파라미터: email

기능:
- 이메일 유효성 검사
- 중복 체크
- D1 DB에 저장 (subscribers 테이블)
- 환영 이메일 전송 (선택)
- JSON 응답

D1 바인딩 설정도 같이.
```

### 블로그 CRUD API

```
블로그 CRUD API를 만들어줘.

엔드포인트:
- GET /api/posts - 목록
- GET /api/posts/[id] - 상세
- POST /api/posts - 작성
- PUT /api/posts/[id] - 수정
- DELETE /api/posts/[id] - 삭제

D1 데이터베이스 사용.
인증 미들웨어 추가 (POST/PUT/DELETE만).
```

### 이미지 업로드 API

```
이미지 업로드 API를 만들어줘.

경로: /api/upload
메서드: POST
파라미터: file (multipart/form-data)

기능:
- 파일 크기 제한 (5MB)
- 이미지 형식만 허용 (jpg, png, gif)
- R2에 저장
- URL 반환

R2 바인딩 설정도 같이.
```

## 체크리스트

Functions 개발 시 확인사항:

- [ ] 적절한 HTTP 메서드를 사용했는가?
- [ ] 오류 처리를 했는가? (try-catch)
- [ ] 적절한 HTTP 상태 코드를 반환하는가?
- [ ] CORS 헤더가 설정되어 있는가?
- [ ] 환경 변수를 올바르게 사용했는가?
- [ ] 파라미터 유효성 검사를 했는가?
- [ ] 인증이 필요한 API에 미들웨어를 적용했는가?

## 자주 하는 실수

### 1. Content-Type 누락

```javascript
// ❌ 잘못된 코드
return new Response(JSON.stringify({ data }));

// ✅ 올바른 코드
return new Response(JSON.stringify({ data }), {
    headers: { 'Content-Type': 'application/json' }
});
```

### 2. 비동기 처리 누락

```javascript
// ❌ 잘못된 코드
export function onRequestPost(context) {
    const data = request.json();  // await 없음
}

// ✅ 올바른 코드
export async function onRequestPost(context) {
    const data = await request.json();
}
```

### 3. 오류 처리 누락

```javascript
// ❌ 잘못된 코드
export async function onRequestGet(context) {
    const data = await fetchData();
    return new Response(JSON.stringify(data));
}

// ✅ 올바른 코드
export async function onRequestGet(context) {
    try {
        const data = await fetchData();
        return new Response(JSON.stringify(data), {
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            error: error.message
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

## 관련 문서

- [프로젝트 시작하기](pages-getting-started.md)
- [환경 변수 및 설정](pages-environment.md)
- [빌드 및 배포](pages-deployment.md)
- [Workers D1 데이터베이스 활용](workers-d1.md)
