# Cloudflare Pages - 라우팅 및 페이지 개발

ViewLogic Router를 활용한 SPA 라우팅과 페이지 개발 방법을 학습합니다.

## ViewLogic Router 기본

### 라우터 구조

`src/logic/router.js`:

```javascript
class Router {
    constructor() {
        this.routes = new Map();
        this.currentPath = '';
    }

    addRoute(path, viewPath, logicPath) {
        this.routes.set(path, { viewPath, logicPath });
    }

    async navigate(path, data = {}) {
        const route = this.routes.get(path) || this.routes.get('/404');
        if (!route) return;

        this.currentPath = path;

        try {
            // 뷰 로드
            const viewRes = await fetch(route.viewPath);
            const viewHTML = await viewRes.text();
            document.getElementById('app').innerHTML = viewHTML;

            // 로직 로드 및 초기화
            if (route.logicPath) {
                const logic = await import(route.logicPath);
                if (logic.init) {
                    await logic.init(data);
                }
            }

            // URL 업데이트
            window.history.pushState({ path, data }, '', path);

            // 스크롤 최상단으로
            window.scrollTo(0, 0);
        } catch (error) {
            console.error('라우팅 오류:', error);
            this.navigate('/404');
        }
    }

    init() {
        // 링크 클릭 이벤트
        document.addEventListener('click', (e) => {
            if (e.target.matches('[data-link]')) {
                e.preventDefault();
                this.navigate(e.target.getAttribute('href'));
            }
        });

        // 뒤로/앞으로 가기
        window.addEventListener('popstate', (e) => {
            if (e.state && e.state.path) {
                this.navigate(e.state.path, e.state.data);
            }
        });

        // 초기 라우팅
        this.navigate(window.location.pathname);
    }
}

export default new Router();
```

### 라우트 등록

```javascript
import router from './router.js';

router.addRoute('/', '/src/views/home.html', '/src/logic/home.js');
router.addRoute('/about', '/src/views/about.html', '/src/logic/about.js');
router.addRoute('/contact', '/src/views/contact.html', '/src/logic/contact.js');
router.addRoute('/blog', '/src/views/blog/list.html', '/src/logic/blog/list.js');
router.addRoute('/404', '/src/views/404.html', null);

router.init();
```

## 기본 페이지 패턴

### 1. 정적 페이지

**뷰:** `src/views/about.html`

```html
<div class="container">
    <h1>회사 소개</h1>
    <p>우리 회사는...</p>
    <img src="/assets/images/team.jpg" alt="팀 사진">
</div>
```

**로직:** `src/logic/about.js`

```javascript
export function init() {
    // 간단한 애니메이션 등
    console.log('소개 페이지 로드됨');
}
```

### 2. 동적 데이터 페이지

**뷰:** `src/views/blog/list.html`

```html
<div class="blog-list">
    <h1>블로그</h1>
    <div id="posts" class="posts-grid"></div>
    <div id="loading">로딩 중...</div>
</div>
```

**로직:** `src/logic/blog/list.js`

```javascript
import { fetchPosts } from '../api.js';

export async function init() {
    showLoading(true);

    try {
        const posts = await fetchPosts();
        renderPosts(posts);
    } catch (error) {
        showError('게시글을 불러올 수 없습니다.');
    } finally {
        showLoading(false);
    }
}

function renderPosts(posts) {
    const container = document.getElementById('posts');
    container.innerHTML = posts.map(post => `
        <article class="post-card">
            <img src="${post.thumbnail}" alt="${post.title}">
            <h2>
                <a href="/blog/${post.slug}" data-link>${post.title}</a>
            </h2>
            <p>${post.excerpt}</p>
            <time datetime="${post.date}">${formatDate(post.date)}</time>
        </article>
    `).join('');
}

function showLoading(show) {
    document.getElementById('loading').style.display = show ? 'block' : 'none';
}

function formatDate(dateString) {
    return new Date(dateString).toLocaleDateString('ko-KR');
}
```

### 3. 폼 처리 페이지

**뷰:** `src/views/contact.html`

```html
<div class="container">
    <h1>연락하기</h1>

    <form id="contactForm">
        <div class="form-group">
            <label for="name">이름</label>
            <input type="text" id="name" name="name" required>
        </div>

        <div class="form-group">
            <label for="email">이메일</label>
            <input type="email" id="email" name="email" required>
        </div>

        <div class="form-group">
            <label for="message">메시지</label>
            <textarea id="message" name="message" required></textarea>
        </div>

        <button type="submit">전송</button>
    </form>

    <div id="result"></div>
</div>
```

**로직:** `src/logic/contact.js`

```javascript
export function init() {
    const form = document.getElementById('contactForm');
    const result = document.getElementById('result');

    form.addEventListener('submit', async (e) => {
        e.preventDefault();

        const formData = new FormData(form);
        const data = Object.fromEntries(formData);

        try {
            const response = await fetch('/api/contact', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data)
            });

            const json = await response.json();

            if (json.success) {
                result.innerHTML = '<p class="success">메시지가 전송되었습니다!</p>';
                form.reset();
            } else {
                result.innerHTML = `<p class="error">${json.message}</p>`;
            }
        } catch (error) {
            result.innerHTML = '<p class="error">전송 실패. 다시 시도해주세요.</p>';
        }
    });
}
```

## 동적 라우팅

### 파라미터 처리

**라우터 개선:**

```javascript
class Router {
    // ... 기존 코드 ...

    matchRoute(path) {
        // 정확히 일치하는 라우트
        if (this.routes.has(path)) {
            return { route: this.routes.get(path), params: {} };
        }

        // 동적 라우트 매칭
        for (const [pattern, route] of this.routes) {
            const regex = new RegExp('^' + pattern.replace(/:\w+/g, '([^/]+)') + '$');
            const match = path.match(regex);

            if (match) {
                const keys = pattern.match(/:\w+/g) || [];
                const params = {};
                keys.forEach((key, i) => {
                    params[key.substring(1)] = match[i + 1];
                });
                return { route, params };
            }
        }

        return { route: this.routes.get('/404'), params: {} };
    }

    async navigate(path, data = {}) {
        const { route, params } = this.matchRoute(path);
        // ... 로직 로드 시 params 전달 ...
        if (logic.init) {
            await logic.init({ ...data, params });
        }
    }
}
```

**라우트 등록:**

```javascript
router.addRoute('/blog/:slug', '/src/views/blog/detail.html', '/src/logic/blog/detail.js');
router.addRoute('/user/:id', '/src/views/user/profile.html', '/src/logic/user/profile.js');
```

**로직에서 파라미터 사용:**

`src/logic/blog/detail.js`:

```javascript
import { fetchPost } from '../api.js';

export async function init({ params }) {
    const { slug } = params;

    try {
        const post = await fetchPost(slug);
        renderPost(post);
    } catch (error) {
        console.error('게시글 로드 실패:', error);
    }
}

function renderPost(post) {
    document.querySelector('h1').textContent = post.title;
    document.querySelector('.content').innerHTML = post.content;
    document.querySelector('time').textContent = post.date;
}
```

## 쿼리 파라미터

### 쿼리 파라미터 파싱

`src/logic/utils.js`:

```javascript
export function getQueryParams() {
    const params = new URLSearchParams(window.location.search);
    return Object.fromEntries(params);
}

export function setQueryParams(params) {
    const url = new URL(window.location);
    Object.entries(params).forEach(([key, value]) => {
        url.searchParams.set(key, value);
    });
    window.history.pushState({}, '', url);
}
```

**사용 예:**

```javascript
import { getQueryParams, setQueryParams } from '../utils.js';

export async function init() {
    const { page = '1', category = 'all' } = getQueryParams();

    // 데이터 로드
    const posts = await fetchPosts({ page, category });
    renderPosts(posts);

    // 페이징 처리
    setupPagination(page);
}

function setupPagination(currentPage) {
    document.querySelectorAll('.page-link').forEach(link => {
        link.addEventListener('click', (e) => {
            e.preventDefault();
            const page = e.target.dataset.page;
            setQueryParams({ page });
            init();  // 재로드
        });
    });
}
```

## 페이지 전환 효과

### CSS 트랜지션

`css/base.css`:

```css
#app {
    animation: fadeIn 0.3s ease-in;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
```

### 프로그레스 바

`src/logic/router.js`:

```javascript
async navigate(path, data = {}) {
    // 프로그레스 바 표시
    showProgress(true);

    try {
        // ... 기존 로드 로직 ...
    } finally {
        // 프로그레스 바 숨김
        showProgress(false);
    }
}

function showProgress(show) {
    const bar = document.getElementById('progress-bar');
    bar.style.display = show ? 'block' : 'none';
}
```

## 실전 프롬프트 예시

### 블로그 상세 페이지 추가

```
블로그 상세 페이지를 만들어줘.

경로: /blog/:slug
뷰: src/views/blog/detail.html
로직: src/logic/blog/detail.js

기능:
- API에서 게시글 데이터 가져오기
- 제목, 내용, 날짜, 작성자 표시
- 이전/다음 게시글 네비게이션
- 공유 버튼 (Twitter, Facebook)

router.js에 동적 라우트 추가.
```

### 검색 페이지 추가

```
검색 페이지를 만들어줘.

경로: /search
쿼리: ?q=검색어&category=카테고리

기능:
- 검색어 입력 폼
- 실시간 검색 (debounce 적용)
- 카테고리 필터
- 결과 목록 표시
- 페이징

URL 쿼리 파라미터로 상태 관리.
```

### 대시보드 페이지 추가

```
사용자 대시보드를 만들어줘.

경로: /dashboard
인증 필요

섹션:
- 프로필 정보
- 최근 활동
- 통계 (그래프)
- 알림

로그인 안 되어 있으면 /login으로 리다이렉트.
```

## 체크리스트

페이지 개발 시 확인사항:

- [ ] 뷰와 로직이 분리되어 있는가?
- [ ] 라우트가 router.js에 등록되어 있는가?
- [ ] data-link 속성을 사용했는가?
- [ ] 동적 파라미터를 올바르게 처리하는가?
- [ ] 오류 처리를 했는가? (try-catch)
- [ ] 로딩 상태를 표시하는가?
- [ ] 페이지 전환 효과가 부드러운가?

## 자주 하는 실수

### 1. data-link 누락

```html
<!-- ❌ 잘못된 코드 -->
<a href="/about">소개</a>

<!-- ✅ 올바른 코드 -->
<a href="/about" data-link>소개</a>
```

### 2. 비동기 처리 누락

```javascript
// ❌ 잘못된 코드
export function init() {
    fetchPosts().then(renderPosts);  // await 없음
}

// ✅ 올바른 코드
export async function init() {
    const posts = await fetchPosts();
    renderPosts(posts);
}
```

### 3. 메모리 누수

```javascript
// ❌ 잘못된 코드
export function init() {
    setInterval(() => {
        updateData();
    }, 1000);  // cleanup 없음
}

// ✅ 올바른 코드
let intervalId;

export function init() {
    intervalId = setInterval(() => {
        updateData();
    }, 1000);
}

export function cleanup() {
    if (intervalId) {
        clearInterval(intervalId);
    }
}
```

## 관련 문서

- [프로젝트 시작하기](pages-getting-started.md)
- [프로젝트 구조](pages-structure.md)
- [Functions 개발](pages-functions.md)
- [정적 에셋 관리](pages-static-assets.md)
