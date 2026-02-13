# Cloudflare Pages - 프로젝트 구조

Cloudflare Pages 프로젝트의 표준 디렉토리 구조와 파일 조직 방법을 학습합니다.

## 표준 프로젝트 구조

### ViewLogic Router 방식

```
myapp/
├── index.html                      # 엔트리 포인트
├── favicon.ico                     # 파비콘
├── robots.txt                      # SEO 설정
├── src/
│   ├── views/                      # HTML 뷰 파일
│   │   ├── layout/                 # 레이아웃 템플릿
│   │   │   └── default.html
│   │   ├── home.html
│   │   ├── about.html
│   │   ├── blog/
│   │   │   ├── list.html
│   │   │   └── detail.html
│   │   └── 404.html
│   ├── logic/                      # JavaScript 로직
│   │   ├── layout/                 # 레이아웃 스크립트
│   │   │   └── default.js
│   │   ├── home.js
│   │   ├── about.js
│   │   └── blog/
│   │       ├── list.js
│   │       └── detail.js
│   └── components/                 # 재사용 컴포넌트 (선택)
├── css/
│   └── base.css                    # 커스텀 스타일 (Bootstrap 보완)
├── assets/
│   └── images/
│       ├── logo.svg
│       └── hero.jpg
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
├── layout/                    # 레이아웃 (헤더/푸터 등 공통 구조)
│   └── default.html
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
├── layout/                    # 레이아웃 스크립트
│   └── default.js
├── home.js                    # 페이지별 로직
├── about.js
└── blog/
    ├── list.js
    └── detail.js
```

**규칙:**
- 뷰 파일과 같은 이름 사용
- 폴더 구조도 동일하게 유지
- `layout/` 폴더는 공통 레이아웃 전용

### CSS 파일

```
css/
└── base.css                   # 커스텀 스타일 (Bootstrap 보완)
```

**스타일 전략:**
- Bootstrap 5 CDN을 기본으로 사용
- `base.css`에는 Bootstrap으로 처리할 수 없는 커스텀 스타일만 작성
- 별도의 variables.css, layout.css 등은 만들지 않음

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

**핵심 요소:**
- `<div id="app">`: ViewLogic이 뷰를 렌더링하는 위치
- Bootstrap 5: 레이아웃/컴포넌트 스타일 담당
- Vue 3: 데이터 바인딩, 이벤트 처리
- ViewLogic Router: 파일 기반 라우팅 자동 처리

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

## 핵심 원칙

1. **뷰 (View):** HTML + Vue 템플릿 문법 (CSS 금지!)
2. **로직 (Logic):** JavaScript (Vue Options API)
3. **스타일 (Style):** Bootstrap 5 + base.css

> 코드 작성법은 [ViewLogic 사용법](pages-viewlogic.md) 참고.

## 체크리스트

프로젝트 구조 확인사항:

- [ ] 뷰와 로직이 분리되어 있는가?
- [ ] layout/default.html, layout/default.js가 있는가?
- [ ] 파일 명명 규칙을 따르는가? (kebab-case)
- [ ] 폴더 구조가 일관성 있는가? (views와 logic 동일 구조)
- [ ] index.html에 Bootstrap + Vue + ViewLogic CDN이 있는가?
- [ ] base.css에 커스텀 스타일만 있는가? (Bootstrap 중복 금지)
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

## 관련 문서

- [프로젝트 시작하기](pages-getting-started.md)
- [프로젝트 구조 표준](project-structure.md)
