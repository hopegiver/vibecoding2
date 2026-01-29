# Cloudflare Pages - 프로젝트 구조

Cloudflare Pages 프로젝트의 표준 디렉토리 구조와 파일 조직 방법을 학습합니다.

## 표준 프로젝트 구조

### ViewLogic Router 방식

```
myapp/
├── index.html                      # 엔트리 포인트
├── favicon.ico                     # 파비콘
├── robots.txt                      # SEO 설정
├── _headers                        # HTTP 헤더 설정
├── _redirects                      # 리다이렉트 규칙
├── src/
│   ├── views/                      # HTML 뷰 파일
│   │   ├── home.html
│   │   ├── about.html
│   │   ├── blog/
│   │   │   ├── list.html
│   │   │   └── detail.html
│   │   └── 404.html
│   ├── logic/                      # JavaScript 로직
│   │   ├── router.js               # 라우터
│   │   ├── api.js                  # API 호출
│   │   ├── utils.js                # 유틸리티
│   │   ├── home.js
│   │   ├── about.js
│   │   └── blog/
│   │       ├── list.js
│   │       └── detail.js
│   └── components/                 # 재사용 컴포넌트
│       ├── header.html
│       ├── footer.html
│       └── card.html
├── css/
│   ├── base.css                    # 기본 스타일
│   ├── variables.css               # CSS 변수
│   ├── layout.css                  # 레이아웃
│   └── components/                 # 컴포넌트 스타일
│       ├── header.css
│       └── card.css
├── assets/
│   ├── images/
│   │   ├── logo.svg
│   │   └── hero.jpg
│   └── fonts/
│       └── custom-font.woff2
├── functions/                      # Cloudflare Pages Functions
│   └── api/
│       ├── contact.js
│       └── newsletter.js
└── .gitignore
```

## 파일 명명 규칙

### 뷰 파일 (HTML)

```
src/views/
├── home.html                  # 단일 단어 소문자
├── about.html
├── contact-us.html            # 여러 단어는 하이픈
├── blog/
│   ├── list.html              # 목록
│   └── detail.html            # 상세
└── user/
    ├── profile.html
    └── settings.html
```

### 로직 파일 (JavaScript)

```
src/logic/
├── router.js                  # 라우터
├── api.js                     # API 클라이언트
├── utils.js                   # 유틸리티
├── home.js                    # 페이지별 로직
└── blog/
    ├── list.js
    └── detail.js
```

**규칙:**
- 뷰 파일과 같은 이름 사용
- 폴더 구조도 동일하게 유지

### CSS 파일

```
css/
├── base.css                   # 기본 스타일
├── variables.css              # CSS 변수
├── layout.css                 # 레이아웃
└── components/
    ├── header.css
    └── card.css
```

## 루트 파일

### index.html

엔트리 포인트, 모든 페이지에서 공통으로 사용:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App</title>
    <link rel="stylesheet" href="/css/base.css">
</head>
<body>
    <nav id="nav"></nav>
    <main id="app"></main>
    <footer id="footer"></footer>

    <script type="module" src="/src/logic/router.js"></script>
</body>
</html>
```

### favicon.ico

루트에 배치:

```
myapp/
├── favicon.ico              # 루트
└── assets/
    └── images/
        └── favicon-192.png  # 다른 크기
```

### robots.txt

SEO 설정:

```
User-agent: *
Allow: /

Sitemap: https://myapp.pages.dev/sitemap.xml
```

## 헤더 및 리다이렉트 설정

### _headers 파일

HTTP 헤더 설정:

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin

/assets/*
  Cache-Control: public, max-age=31536000, immutable

/css/*
  Cache-Control: public, max-age=31536000

/src/*
  Cache-Control: public, max-age=3600
```

### _redirects 파일

리다이렉트 규칙:

```
/old-page    /new-page    301
/blog/:slug  /articles/:slug  301
/api/*       https://api.example.com/:splat  200
```

## 뷰와 로직 분리

### 기본 원칙

1. **뷰 (View):** HTML만 담당
2. **로직 (Logic):** JavaScript로 동작 제어
3. **스타일 (Style):** CSS로 디자인

### 예제

**뷰:** `src/views/blog/list.html`

```html
<div class="blog-list">
    <h1>블로그</h1>
    <div id="posts"></div>
    <div id="pagination"></div>
</div>
```

**로직:** `src/logic/blog/list.js`

```javascript
import { fetchPosts } from '../api.js';

export async function init() {
    const posts = await fetchPosts();
    renderPosts(posts);
}

function renderPosts(posts) {
    const container = document.getElementById('posts');
    container.innerHTML = posts.map(post => `
        <article class="post-card">
            <h2><a href="/blog/${post.slug}" data-link>${post.title}</a></h2>
            <p>${post.excerpt}</p>
        </article>
    `).join('');
}
```

## 컴포넌트 구조

### 재사용 가능한 컴포넌트

`src/components/header.html`:

```html
<header class="site-header">
    <div class="container">
        <a href="/" class="logo" data-link>My App</a>
        <nav>
            <a href="/" data-link>홈</a>
            <a href="/about" data-link>소개</a>
            <a href="/blog" data-link>블로그</a>
        </nav>
    </div>
</header>
```

### 컴포넌트 로드

`src/logic/router.js`:

```javascript
async function loadComponent(id, path) {
    const response = await fetch(path);
    const html = await response.text();
    document.getElementById(id).innerHTML = html;
}

// 초기화 시 컴포넌트 로드
loadComponent('nav', '/src/components/header.html');
loadComponent('footer', '/src/components/footer.html');
```

## Functions 디렉토리

### API 엔드포인트

`functions/api/contact.js`:

```javascript
export async function onRequestPost(context) {
    const { request } = context;
    const data = await request.json();

    // 처리 로직

    return new Response(JSON.stringify({ success: true }), {
        headers: { 'Content-Type': 'application/json' }
    });
}
```

**URL:** `https://myapp.pages.dev/api/contact`

## 환경별 설정

### 개발 환경

```
.env.development
```

```
API_URL=http://localhost:8787
DEBUG=true
```

### 프로덕션 환경

```
.env.production
```

```
API_URL=https://api.myapp.com
DEBUG=false
```

## Claude Code와 함께 사용하기

### 프로젝트 구조 생성 요청

```
Cloudflare Pages 프로젝트 구조를 만들어줘.

ViewLogic Router 방식
폴더: src/views, src/logic, css, assets
루트 파일: index.html, favicon.ico, robots.txt

기본 페이지:
- 홈 (/)
- 소개 (/about)
- 연락처 (/contact)
- 404

각 페이지마다 뷰와 로직 분리.
```

### 컴포넌트 추가 요청

```
재사용 가능한 헤더 컴포넌트를 만들어줘.

위치: src/components/header.html
내용: 로고, 메뉴 (홈, 소개, 블로그)
반응형 디자인 (모바일 햄버거 메뉴)

router.js에서 자동 로드.
```

### 페이지 추가 요청

```
블로그 섹션을 추가해줘.

페이지:
1. /blog - 목록 (src/views/blog/list.html)
2. /blog/:slug - 상세 (src/views/blog/detail.html)

로직:
- src/logic/blog/list.js
- src/logic/blog/detail.js

API에서 데이터 가져오기.
router.js에 라우트 추가.
```

## 체크리스트

프로젝트 구조 확인사항:

- [ ] 뷰와 로직이 분리되어 있는가?
- [ ] 파일 명명 규칙을 따르는가?
- [ ] 폴더 구조가 일관성 있는가?
- [ ] 컴포넌트가 재사용 가능한가?
- [ ] CSS가 모듈화되어 있는가?
- [ ] Functions이 /functions에 있는가?
- [ ] 루트 파일이 올바른 위치에 있는가?

## 자주 하는 실수

### 1. 뷰와 로직 경로 불일치

```
❌ 잘못된 구조:
src/views/blog-list.html
src/logic/blogList.js

✅ 올바른 구조:
src/views/blog/list.html
src/logic/blog/list.js
```

### 2. 루트 파일 위치 오류

```
❌ 잘못된 위치:
src/index.html

✅ 올바른 위치:
index.html (루트)
```

### 3. 컴포넌트 미분리

```
❌ 잘못된 방식:
각 뷰마다 헤더/푸터 중복

✅ 올바른 방식:
src/components/header.html 공통 사용
```

## 관련 문서

- [프로젝트 시작하기](pages-getting-started.md)
- [라우팅 및 페이지 개발](pages-routing.md)
- [Functions 개발](pages-functions.md)
- [프로젝트 구조 표준](project-structure.md)
