# Cloudflare Pages - ViewLogic 사용법

ViewLogic Router를 활용한 SPA 개발 방법을 학습합니다.

## ViewLogic이란?

파일 기반 라우팅을 제공하는 SPA 라우터입니다. Vue 3 + Bootstrap 5와 함께 사용하며, 빌드 없이 바로 개발할 수 있습니다.

**핵심 특징:**
- 파일 이름 = 라우트 (`home.html` → `/#/home`)
- View/Logic 자동 매칭 (`views/home.html` ↔ `logic/home.js`)
- Vue 3 Options API 기반 (data, methods, computed, watch)
- 레이아웃 시스템 (`layout/default.html`)
- 내장 API 클라이언트 (`this.$api`)
- 인증, 다국어, 캐싱 지원

## 설치 및 설정

### index.html

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App</title>

    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- 커스텀 스타일 -->
    <link rel="stylesheet" href="/css/base.css">
</head>
<body>
    <div id="app"></div>

    <!-- Bootstrap 5 JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <!-- Vue 3 -->
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <!-- ViewLogic Router -->
    <script src="https://cdn.jsdelivr.net/npm/viewlogic@latest/dist/viewlogic-router.min.js"></script>
    <script>
        const router = new ViewLogicRouter({
            basePath: '/',
            srcPath: '/src',
            mode: 'hash'
        });
    </script>
</body>
</html>
```

### 설정 옵션

```javascript
const router = new ViewLogicRouter({
    // 기본
    basePath: '/',
    srcPath: '/src',
    mode: 'hash',               // 'hash' 또는 'history'

    // 레이아웃
    useLayout: true,
    defaultLayout: 'default',

    // 캐싱
    cacheMode: 'memory',        // 'memory', 'sessionStorage', 'localStorage', 'none'
    cacheTTL: 300000,           // 5분

    // 인증
    authEnabled: false,
    loginRoute: 'login',
    protectedRoutes: [],

    // 다국어
    useI18n: false,
    defaultLanguage: 'ko',

    // 로깅
    logLevel: 'info'            // 'debug', 'info', 'warn', 'error'
});
```

## 라우팅

### 파일 기반 라우트

파일 이름이 곧 라우트입니다:

| 파일 경로 | URL |
|---|---|
| `views/home.html` + `logic/home.js` | `/#/home` |
| `views/about.html` + `logic/about.js` | `/#/about` |
| `views/blog/list.html` + `logic/blog/list.js` | `/#/blog/list` |
| `views/admin/dashboard.html` | `/#/admin/dashboard` |

### 페이지 이동

```javascript
// HTML에서
<a href="#/about">소개 페이지로</a>

// JavaScript에서
this.navigateTo('/about');

// 파라미터와 함께
this.navigateTo('/users', { id: 123, tab: 'profile' });
// 결과: /#/users?id=123&tab=profile
```

### 파라미터 받기

URL: `/#/users?id=123&tab=profile`

```javascript
export default {
    name: 'Users',
    data() {
        return {
            userId: this.getParam('id'),          // '123'
            tab: this.getParam('tab', 'default')  // 'profile' (기본값 지정 가능)
        }
    },
    methods: {
        loadData() {
            const allParams = this.getParams();   // { id: '123', tab: 'profile' }
        }
    }
}
```

## 뷰 작성법 (HTML)

### 기본 규칙

- HTML + Vue 템플릿 문법만 사용
- `<style>`, `<script>` 태그 금지
- Bootstrap 클래스 최대한 활용

### Vue 템플릿 문법

```html
<!-- 데이터 바인딩 -->
<h1>{{ title }}</h1>
<p>{{ message }}</p>

<!-- 조건부 렌더링 -->
<div v-if="loading">로딩 중...</div>
<div v-else-if="error">오류 발생</div>
<div v-else>{{ content }}</div>

<!-- 반복 렌더링 -->
<div v-for="item in items" :key="item.id">
    {{ item.name }}
</div>

<!-- 이벤트 처리 -->
<button @click="handleClick">클릭</button>
<form @submit.prevent="handleSubmit">...</form>

<!-- 양방향 바인딩 -->
<input v-model="keyword" placeholder="검색어">

<!-- 속성 바인딩 -->
<img :src="imageUrl" :alt="title">
<a :href="'#/detail?id=' + item.id">상세</a>
```

### 예제: 목록 페이지

`src/views/blog/list.html`:

```html
<div class="row">
    <div class="col-12 mb-4">
        <h1>블로그</h1>
    </div>

    <div v-if="loading" class="col-12 text-center">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">로딩 중...</span>
        </div>
    </div>

    <div v-else class="col-md-4 mb-3" v-for="post in posts" :key="post.id">
        <div class="card h-100">
            <div class="card-body">
                <h5 class="card-title">{{ post.title }}</h5>
                <p class="card-text text-muted">{{ post.excerpt }}</p>
            </div>
            <div class="card-footer">
                <a :href="'#/blog/detail?slug=' + post.slug" class="btn btn-sm btn-primary">
                    자세히 보기
                </a>
            </div>
        </div>
    </div>
</div>
```

## 로직 작성법 (JavaScript)

### 기본 구조

```javascript
export default {
    name: 'PageName',
    layout: 'default',          // 레이아웃 지정

    data() {
        return {
            // 반응형 데이터
        }
    },

    computed: {
        // 계산된 속성
    },

    watch: {
        // 데이터 감시
    },

    async mounted() {
        // DOM 마운트 후 실행 (데이터 로딩 등)
    },

    beforeUnmount() {
        // 정리 작업
    },

    methods: {
        // 메서드
    }
}
```

### 예제: 목록 로직

`src/logic/blog/list.js`:

```javascript
export default {
    name: 'BlogList',
    layout: 'default',
    data() {
        return {
            posts: [],
            loading: false
        }
    },
    async mounted() {
        await this.loadPosts();
    },
    methods: {
        async loadPosts() {
            this.loading = true;
            try {
                const response = await this.$api.get('/api/posts');
                this.posts = response.data;
            } catch (error) {
                console.error('로딩 실패:', error);
            } finally {
                this.loading = false;
            }
        }
    }
}
```

### 데이터 자동 로딩 (dataURL)

간단한 GET 조회는 `dataURL`로 자동 처리:

```javascript
// 기본 사용 (문자열)
export default {
    name: 'Users',
    layout: 'default',
    dataURL: '/api/users',      // GET 요청 자동 실행
    data() {
        return {
            users: []           // API 응답으로 자동 채워짐
        }
    },
    mounted() {
        console.log(this.users); // 이미 데이터가 있음
    }
}
```

```javascript
// 파라미터 치환
export default {
    name: 'UserDetail',
    layout: 'default',
    dataURL: '/api/users/{id}', // {id}는 쿼리 파라미터에서 자동 치환
    data() {
        return {
            user: null
        }
    }
}
// 접근: /#/user-detail?id=123 → GET /api/users/123
```

## API 호출

### this.$api 메서드

```javascript
// GET
const users = await this.$api.get('/api/users');
const filtered = await this.$api.get('/api/users', {
    params: { role: 'admin', page: 1 }
});

// POST
const created = await this.$api.post('/api/users', {
    name: 'John',
    email: 'john@example.com'
});

// PUT
const updated = await this.$api.put('/api/users/123', { name: 'John Doe' });

// DELETE
const deleted = await this.$api.delete('/api/users/123');
```

### 에러 처리

```javascript
methods: {
    async fetchData() {
        try {
            const response = await this.$api.get('/api/data');
            this.data = response.data;
        } catch (error) {
            if (error.response) {
                console.error('서버 에러:', error.response.status);
            } else {
                console.error('네트워크 에러:', error.message);
            }
        }
    }
}
```

## 폼 처리

### 기본 폼

```html
<form @submit.prevent="handleSubmit">
    <div class="mb-3">
        <label for="name" class="form-label">이름</label>
        <input v-model="form.name" type="text" class="form-control" id="name" required>
    </div>
    <div class="mb-3">
        <label for="email" class="form-label">이메일</label>
        <input v-model="form.email" type="email" class="form-control" id="email" required>
    </div>
    <button type="submit" class="btn btn-primary">저장</button>
</form>
```

```javascript
export default {
    name: 'Contact',
    layout: 'default',
    data() {
        return {
            form: { name: '', email: '' }
        }
    },
    methods: {
        async handleSubmit() {
            const response = await this.$api.post('/api/contact', this.form);
            if (response.success) {
                alert('전송 완료!');
                this.navigateTo('/home');
            }
        }
    }
}
```

### 선언적 폼 (자동 처리)

```html
<form
    action="/api/users/{userId}"
    method="PUT"
    data-success="handleSuccess"
    data-error="handleError"
    data-redirect="users">

    <input name="name" v-model="form.name" class="form-control">
    <button type="submit" class="btn btn-primary">수정</button>
</form>
```

폼 속성:
- `action` - API 엔드포인트 (`{param}`은 data에서 자동 치환)
- `method` - HTTP 메서드
- `data-success` - 성공 시 호출할 메서드명
- `data-error` - 실패 시 호출할 메서드명
- `data-redirect` - 성공 후 이동할 라우트

## 레이아웃

### 레이아웃 생성

**뷰:** `src/views/layout/default.html`

```html
<div class="layout-default">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="#/home">My App</a>
            <button class="navbar-toggler" type="button"
                    data-bs-toggle="collapse" data-bs-target="#navMenu">
                <span class="navbar-toggler-icon"></span>
            </button>
            <div class="collapse navbar-collapse" id="navMenu">
                <ul class="navbar-nav">
                    <li class="nav-item"><a class="nav-link" href="#/home">홈</a></li>
                    <li class="nav-item"><a class="nav-link" href="#/about">소개</a></li>
                    <li class="nav-item"><a class="nav-link" href="#/blog/list">블로그</a></li>
                </ul>
            </div>
        </div>
    </nav>

    <main class="container mt-4">
        {{ content }}
    </main>

    <footer class="bg-light text-center py-3 mt-5">
        <p class="mb-0">&copy; 2025 My App</p>
    </footer>
</div>
```

`{{ content }}`에 각 페이지의 뷰가 렌더링됩니다.

**로직:** `src/logic/layout/default.js` (선택)

```javascript
export default {
    name: 'defaultLayout',
    data() {
        return {}
    },
    mounted() {
        // 레이아웃 초기화
    }
}
```

### 레이아웃 사용

```javascript
// 기본 레이아웃 사용
export default {
    name: 'Home',
    layout: 'default',
    // ...
}

// 관리자 레이아웃 사용
export default {
    name: 'AdminDashboard',
    layout: 'admin',
    // ...
}

// 레이아웃 없이 (로그인 등)
export default {
    name: 'Login',
    layout: null,
    // ...
}
```

## 인증

### 설정

```javascript
const router = new ViewLogicRouter({
    authEnabled: true,
    loginRoute: 'login',
    protectedRoutes: ['profile', 'admin/*'],
    authStorage: 'localStorage'
});
```

### 로그인/로그아웃

```javascript
// 로그인
export default {
    name: 'Login',
    layout: null,
    data() {
        return { username: '', password: '', error: '' }
    },
    methods: {
        async handleLogin() {
            try {
                const response = await this.$api.post('/api/login', {
                    username: this.username,
                    password: this.password
                });
                this.setToken(response.token);
                this.navigateTo('/home');
            } catch (error) {
                this.error = '로그인 실패';
            }
        }
    }
}
```

```javascript
// 인증 상태 확인
mounted() {
    if (this.isAuth()) {
        this.loadProfile();
    }
}

// 로그아웃
methods: {
    handleLogout() {
        this.logout();  // 자동으로 login 페이지로 이동
    }
}
```

인증 활성화 시 모든 API 요청에 `Authorization: Bearer TOKEN` 헤더가 자동 추가됩니다.

## 컴포넌트

### 컴포넌트 생성

`src/components/Card.js`:

```javascript
export default {
    name: 'Card',
    props: {
        title: String,
        text: String,
        link: String
    },
    template: `
        <div class="card h-100">
            <div class="card-body">
                <h5 class="card-title">{{ title }}</h5>
                <p class="card-text">{{ text }}</p>
                <a v-if="link" :href="link" class="btn btn-primary btn-sm">자세히</a>
            </div>
        </div>
    `
}
```

### 페이지에서 사용

```javascript
export default {
    name: 'Home',
    layout: 'default',
    components: ['Card'],    // 컴포넌트 이름만 명시
    data() {
        return {
            items: []
        }
    }
}
```

```html
<div class="row">
    <div class="col-md-4" v-for="item in items" :key="item.id">
        <Card :title="item.title" :text="item.desc" :link="'#/detail?id=' + item.id" />
    </div>
</div>
```

## 유용한 빌트인 기능

### Computed & Watch

```javascript
export default {
    name: 'Cart',
    data() {
        return {
            items: [],
            keyword: ''
        }
    },
    computed: {
        totalPrice() {
            return this.items.reduce((sum, item) => sum + item.price * item.qty, 0);
        }
    },
    watch: {
        keyword(newVal) {
            this.search(newVal);  // keyword 변경 시 자동 검색
        }
    }
}
```

### 전역 상태

```javascript
// 상태 저장
this.$state.set('user', { name: 'John', role: 'admin' });

// 상태 읽기
const user = this.$state.get('user');

// 변경 감지
this.$state.watch('user', (newUser) => {
    this.user = newUser;
});
```

### 프로그레스 바

0.3초 이상 로딩 시 자동 표시. 커스터마이즈:

```css
#viewlogic-progress-bar {
    background-color: #0d6efd !important;
    height: 3px !important;
}
```

## 체크리스트

ViewLogic 개발 시 확인사항:

- [ ] index.html에 Bootstrap + Vue + ViewLogic CDN이 있는가?
- [ ] `<div id="app"></div>`이 있는가?
- [ ] views와 logic 폴더 구조가 동일한가?
- [ ] HTML 파일에 `<style>`, `<script>` 태그가 없는가?
- [ ] 로직에서 `layout: 'default'` 를 지정했는가?
- [ ] `this.$api`로 API를 호출하는가?
- [ ] Bootstrap 클래스를 최대한 활용하고 있는가?

## 관련 문서

- [프로젝트 구조](pages-structure.md)
- [프로젝트 시작하기](pages-getting-started.md)
