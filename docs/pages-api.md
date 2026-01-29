# Cloudflare Pages - API 엔드포인트 개발

Pages Functions를 활용한 RESTful API 개발과 프론트엔드 연동 방법을 학습합니다.

## API 클라이언트 구조

### api.js 모듈

`src/logic/api.js`:

```javascript
const API_BASE_URL = '/api';

class ApiClient {
    async request(endpoint, options = {}) {
        const url = `${API_BASE_URL}${endpoint}`;

        const config = {
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            ...options
        };

        try {
            const response = await fetch(url, config);
            const data = await response.json();

            if (!response.ok) {
                throw new Error(data.message || '요청 실패');
            }

            return data;
        } catch (error) {
            console.error('API 오류:', error);
            throw error;
        }
    }

    async get(endpoint, params = {}) {
        const query = new URLSearchParams(params).toString();
        const url = query ? `${endpoint}?${query}` : endpoint;
        return this.request(url, { method: 'GET' });
    }

    async post(endpoint, body) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(body)
        });
    }

    async put(endpoint, body) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(body)
        });
    }

    async delete(endpoint) {
        return this.request(endpoint, {
            method: 'DELETE'
        });
    }
}

export const api = new ApiClient();

// 편의 함수
export async function fetchPosts(params) {
    return api.get('/posts', params);
}

export async function fetchPost(id) {
    return api.get(`/posts/${id}`);
}

export async function createPost(data) {
    return api.post('/posts', data);
}

export async function updatePost(id, data) {
    return api.put(`/posts/${id}`, data);
}

export async function deletePost(id) {
    return api.delete(`/posts/${id}`);
}
```

## CRUD API 구현

### 목록 조회 (GET)

**Backend:** `functions/api/posts/index.js`

```javascript
export async function onRequestGet(context) {
    const { request, env } = context;
    const { searchParams } = new URL(request.url);

    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const offset = (page - 1) * limit;

    const { results: posts } = await env.DB.prepare(
        'SELECT * FROM posts ORDER BY created_at DESC LIMIT ? OFFSET ?'
    ).bind(limit, offset).all();

    const { results: [{ total }] } = await env.DB.prepare(
        'SELECT COUNT(*) as total FROM posts'
    ).all();

    return new Response(JSON.stringify({
        success: true,
        data: posts,
        pagination: {
            page,
            limit,
            total,
            totalPages: Math.ceil(total / limit)
        }
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

**Frontend:** `src/logic/blog/list.js`

```javascript
import { fetchPosts } from '../api.js';

export async function init() {
    const page = new URLSearchParams(window.location.search).get('page') || '1';

    try {
        const { data: posts, pagination } = await fetchPosts({ page, limit: 10 });
        renderPosts(posts);
        renderPagination(pagination);
    } catch (error) {
        showError('게시글을 불러올 수 없습니다.');
    }
}

function renderPosts(posts) {
    const container = document.getElementById('posts');
    container.innerHTML = posts.map(post => `
        <article class="post-card">
            <h2><a href="/blog/${post.id}" data-link>${post.title}</a></h2>
            <p>${post.excerpt}</p>
        </article>
    `).join('');
}

function renderPagination(pagination) {
    const { page, totalPages } = pagination;
    const container = document.getElementById('pagination');

    const pages = [];
    for (let i = 1; i <= totalPages; i++) {
        const active = i === page ? 'active' : '';
        pages.push(`
            <a href="/blog?page=${i}" data-link class="page-link ${active}">
                ${i}
            </a>
        `);
    }

    container.innerHTML = pages.join('');
}
```

### 상세 조회 (GET)

**Backend:** `functions/api/posts/[id].js`

```javascript
export async function onRequestGet(context) {
    const { params, env } = context;
    const postId = params.id;

    const { results } = await env.DB.prepare(
        'SELECT * FROM posts WHERE id = ?'
    ).bind(postId).all();

    if (results.length === 0) {
        return new Response(JSON.stringify({
            success: false,
            message: '게시글을 찾을 수 없습니다.'
        }), {
            status: 404,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    return new Response(JSON.stringify({
        success: true,
        data: results[0]
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

**Frontend:** `src/logic/blog/detail.js`

```javascript
import { fetchPost } from '../api.js';

export async function init({ params }) {
    const postId = params.id;

    try {
        const { data: post } = await fetchPost(postId);
        renderPost(post);
    } catch (error) {
        showError('게시글을 불러올 수 없습니다.');
    }
}

function renderPost(post) {
    document.querySelector('h1').textContent = post.title;
    document.querySelector('.content').innerHTML = post.content;
    document.querySelector('time').textContent = new Date(post.created_at).toLocaleDateString();
}
```

### 생성 (POST)

**Backend:** `functions/api/posts/index.js`

```javascript
export async function onRequestPost(context) {
    const { request, env } = context;

    try {
        const data = await request.json();

        // 유효성 검사
        if (!data.title || !data.content) {
            return new Response(JSON.stringify({
                success: false,
                message: '제목과 내용을 입력하세요.'
            }), {
                status: 400,
                headers: { 'Content-Type': 'application/json' }
            });
        }

        // DB 저장
        const { success, meta } = await env.DB.prepare(
            'INSERT INTO posts (title, content, created_at) VALUES (?, ?, ?)'
        ).bind(data.title, data.content, new Date().toISOString()).run();

        if (!success) {
            throw new Error('저장 실패');
        }

        return new Response(JSON.stringify({
            success: true,
            message: '저장되었습니다.',
            id: meta.last_row_id
        }), {
            status: 201,
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            success: false,
            message: error.message
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

**Frontend:** `src/logic/blog/create.js`

```javascript
import { createPost } from '../api.js';
import router from '../router.js';

export function init() {
    const form = document.getElementById('postForm');

    form.addEventListener('submit', async (e) => {
        e.preventDefault();

        const formData = new FormData(form);
        const data = {
            title: formData.get('title'),
            content: formData.get('content')
        };

        try {
            const { id } = await createPost(data);
            alert('저장되었습니다!');
            router.navigate(`/blog/${id}`);
        } catch (error) {
            alert('저장 실패: ' + error.message);
        }
    });
}
```

### 수정 (PUT)

**Backend:** `functions/api/posts/[id].js`

```javascript
export async function onRequestPut(context) {
    const { request, params, env } = context;
    const postId = params.id;

    try {
        const data = await request.json();

        const { success } = await env.DB.prepare(
            'UPDATE posts SET title = ?, content = ?, updated_at = ? WHERE id = ?'
        ).bind(data.title, data.content, new Date().toISOString(), postId).run();

        if (!success) {
            throw new Error('수정 실패');
        }

        return new Response(JSON.stringify({
            success: true,
            message: '수정되었습니다.'
        }), {
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            success: false,
            message: error.message
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

### 삭제 (DELETE)

**Backend:** `functions/api/posts/[id].js`

```javascript
export async function onRequestDelete(context) {
    const { params, env } = context;
    const postId = params.id;

    try {
        const { success } = await env.DB.prepare(
            'DELETE FROM posts WHERE id = ?'
        ).bind(postId).run();

        if (!success) {
            throw new Error('삭제 실패');
        }

        return new Response(JSON.stringify({
            success: true,
            message: '삭제되었습니다.'
        }), {
            headers: { 'Content-Type': 'application/json' }
        });
    } catch (error) {
        return new Response(JSON.stringify({
            success: false,
            message: error.message
        }), {
            status: 500,
            headers: { 'Content-Type': 'application/json' }
        });
    }
}
```

## 인증 처리

### JWT 토큰 생성

**Backend:** `functions/api/auth/login.js`

```javascript
import jwt from '@tsndr/cloudflare-worker-jwt';

export async function onRequestPost(context) {
    const { request, env } = context;
    const { email, password } = await request.json();

    // 사용자 확인
    const { results: users } = await env.DB.prepare(
        'SELECT * FROM users WHERE email = ? AND password = ?'
    ).bind(email, password).all();

    if (users.length === 0) {
        return new Response(JSON.stringify({
            success: false,
            message: '이메일 또는 비밀번호가 일치하지 않습니다.'
        }), {
            status: 401,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // JWT 생성
    const token = await jwt.sign({
        userId: users[0].id,
        email: users[0].email
    }, env.JWT_SECRET);

    return new Response(JSON.stringify({
        success: true,
        token
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

### 인증 미들웨어

`functions/api/_middleware.js`:

```javascript
import jwt from '@tsndr/cloudflare-worker-jwt';

export async function onRequest(context) {
    const { request, next, env } = context;

    // 인증이 필요 없는 경로
    const publicPaths = ['/api/auth/login', '/api/auth/register'];
    if (publicPaths.includes(new URL(request.url).pathname)) {
        return next();
    }

    // 토큰 확인
    const token = request.headers.get('Authorization')?.replace('Bearer ', '');

    if (!token) {
        return new Response(JSON.stringify({
            success: false,
            message: '인증이 필요합니다.'
        }), {
            status: 401,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // 토큰 검증
    const isValid = await jwt.verify(token, env.JWT_SECRET);

    if (!isValid) {
        return new Response(JSON.stringify({
            success: false,
            message: '유효하지 않은 토큰입니다.'
        }), {
            status: 403,
            headers: { 'Content-Type': 'application/json' }
        });
    }

    // 토큰 데이터를 context에 추가
    const { payload } = jwt.decode(token);
    context.data.user = payload;

    return next();
}
```

### Frontend에서 토큰 사용

`src/logic/api.js`:

```javascript
class ApiClient {
    constructor() {
        this.token = localStorage.getItem('auth_token');
    }

    setToken(token) {
        this.token = token;
        localStorage.setItem('auth_token', token);
    }

    clearToken() {
        this.token = null;
        localStorage.removeItem('auth_token');
    }

    async request(endpoint, options = {}) {
        const config = {
            headers: {
                'Content-Type': 'application/json',
                ...(this.token && { 'Authorization': `Bearer ${this.token}` }),
                ...options.headers
            },
            ...options
        };

        // ... 나머지 코드 ...
    }
}

export const api = new ApiClient();

// 로그인
export async function login(email, password) {
    const { token } = await api.post('/auth/login', { email, password });
    api.setToken(token);
    return token;
}

// 로그아웃
export function logout() {
    api.clearToken();
}
```

## 실전 프롬프트 예시

### 댓글 API 추가

```
댓글 CRUD API를 만들어줘.

엔드포인트:
- GET /api/posts/:id/comments - 댓글 목록
- POST /api/posts/:id/comments - 댓글 작성
- DELETE /api/comments/:id - 댓글 삭제

D1 테이블: comments (id, post_id, user_id, content, created_at)

인증 필요 (POST, DELETE).
Frontend api.js에 편의 함수 추가.
```

### 검색 API 추가

```
검색 API를 만들어줘.

엔드포인트: GET /api/search
쿼리: ?q=검색어&type=posts|users

기능:
- 제목/내용 검색 (posts)
- 이름/이메일 검색 (users)
- D1 LIKE 쿼리 사용
- 페이징 지원

Frontend에서 실시간 검색 (debounce 300ms).
```

### 파일 업로드 API

```
파일 업로드 API를 만들어줘.

엔드포인트: POST /api/upload
파라미터: file (multipart/form-data)

기능:
- 5MB 제한
- 이미지만 허용 (jpg, png, gif, webp)
- R2 버킷에 저장
- URL 반환 (https://r2.example.com/...)

Frontend에서 드래그앤드롭 지원.
```

## 체크리스트

API 개발 시 확인사항:

- [ ] API 클라이언트가 모듈화되어 있는가?
- [ ] 오류 처리가 적절한가?
- [ ] 인증이 필요한 API에 미들웨어를 적용했는가?
- [ ] 토큰을 안전하게 저장하고 있는가? (localStorage)
- [ ] CORS 헤더가 설정되어 있는가?
- [ ] 파라미터 유효성 검사를 했는가?
- [ ] 적절한 HTTP 상태 코드를 반환하는가?

## 자주 하는 실수

### 1. 토큰 누락

```javascript
// ❌ 잘못된 코드
async request(endpoint, options) {
    // Authorization 헤더 없음
    return fetch(endpoint, options);
}

// ✅ 올바른 코드
async request(endpoint, options) {
    const config = {
        ...options,
        headers: {
            ...options.headers,
            ...(this.token && { 'Authorization': `Bearer ${this.token}` })
        }
    };
    return fetch(endpoint, config);
}
```

### 2. 에러 처리 누락

```javascript
// ❌ 잘못된 코드
export async function init() {
    const posts = await fetchPosts();
    renderPosts(posts);
}

// ✅ 올바른 코드
export async function init() {
    try {
        const posts = await fetchPosts();
        renderPosts(posts);
    } catch (error) {
        showError('게시글을 불러올 수 없습니다.');
    }
}
```

### 3. Content-Type 누락

```javascript
// ❌ 잘못된 코드 (Backend)
return new Response(JSON.stringify({ data }));

// ✅ 올바른 코드
return new Response(JSON.stringify({ data }), {
    headers: { 'Content-Type': 'application/json' }
});
```

## 관련 문서

- [Functions 개발](pages-functions.md)
- [환경 변수 및 설정](pages-environment.md)
- [Workers D1 데이터베이스 활용](workers-d1.md)
- [보안 가이드라인](security.md)
