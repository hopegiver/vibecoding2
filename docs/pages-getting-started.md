# Cloudflare Pages - 프로젝트 시작하기

Cloudflare Pages로 정적 사이트 또는 ViewLogic Router 기반 SPA 프로젝트를 시작하는 방법을 안내합니다.

## 전제조건

- Node.js 18 이상
- Git
- Cloudflare 계정
- VSCode + Claude Code

## 1. 프로젝트 초기 설정

### Claude에게 요청하기

```
Cloudflare Pages 프로젝트를 새로 만들어줘.
프로젝트 이름: myapp
구조: ViewLogic Router 방식

구조:
- index.html (루트)
- src/views/ (HTML 뷰)
- src/logic/ (JS 로직)
- css/ (스타일)
```

### 생성되는 기본 구조

```
myapp/
├── index.html                 # 엔트리 포인트
├── favicon.ico                # 파비콘
├── robots.txt                 # SEO
├── src/
│   ├── views/                 # HTML 뷰
│   │   ├── home.html
│   │   └── about.html
│   └── logic/                 # JavaScript 로직
│       ├── router.js          # 라우터
│       ├── home.js
│       └── about.js
├── css/
│   └── base.css               # 기본 스타일
└── .gitignore
```

## 2. index.html 설정

### 기본 index.html

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
    <nav>
        <a href="/" data-link>홈</a>
        <a href="/about" data-link>소개</a>
    </nav>

    <main id="app"></main>

    <script type="module" src="/src/logic/router.js"></script>
</body>
</html>
```

**핵심 요소:**
- `<main id="app">`: 뷰가 렌더링되는 위치
- `data-link`: 라우터가 처리하는 링크
- `type="module"`: ES6 모듈 사용

## 3. ViewLogic Router 설정

### router.js 구현

`src/logic/router.js`:

```javascript
class Router {
    constructor() {
        this.routes = {};
        this.currentPath = '';
    }

    // 라우트 등록
    addRoute(path, viewPath, logicPath) {
        this.routes[path] = { viewPath, logicPath };
    }

    // 라우팅 처리
    async navigate(path) {
        const route = this.routes[path] || this.routes['/404'];
        if (!route) {
            document.getElementById('app').innerHTML = '<h1>404 Not Found</h1>';
            return;
        }

        this.currentPath = path;

        try {
            // 뷰 로드
            const viewResponse = await fetch(route.viewPath);
            const viewHTML = await viewResponse.text();
            document.getElementById('app').innerHTML = viewHTML;

            // 로직 로드
            if (route.logicPath) {
                const logic = await import(route.logicPath);
                if (logic.init) {
                    logic.init();
                }
            }

            // URL 업데이트
            window.history.pushState({}, '', path);
        } catch (error) {
            console.error('라우팅 오류:', error);
        }
    }

    // 이벤트 리스너 설정
    init() {
        // 링크 클릭 이벤트
        document.addEventListener('click', (e) => {
            if (e.target.matches('[data-link]')) {
                e.preventDefault();
                this.navigate(e.target.getAttribute('href'));
            }
        });

        // 뒤로가기/앞으로가기
        window.addEventListener('popstate', () => {
            this.navigate(window.location.pathname);
        });

        // 초기 라우팅
        this.navigate(window.location.pathname);
    }
}

// 라우트 설정
const router = new Router();
router.addRoute('/', '/src/views/home.html', '/src/logic/home.js');
router.addRoute('/about', '/src/views/about.html', '/src/logic/about.js');
router.addRoute('/404', '/src/views/404.html', null);

// 라우터 초기화
router.init();

export default router;
```

## 4. 첫 페이지 만들기

### 뷰 (View)

`src/views/home.html`:

```html
<div class="container">
    <h1>환영합니다!</h1>
    <p>Cloudflare Pages로 만든 웹사이트입니다.</p>

    <div id="counter">
        <p>카운터: <span id="count">0</span></p>
        <button id="increment">증가</button>
        <button id="decrement">감소</button>
    </div>
</div>
```

### 로직 (Logic)

`src/logic/home.js`:

```javascript
export function init() {
    let count = 0;
    const countEl = document.getElementById('count');
    const incrementBtn = document.getElementById('increment');
    const decrementBtn = document.getElementById('decrement');

    incrementBtn.addEventListener('click', () => {
        count++;
        countEl.textContent = count;
    });

    decrementBtn.addEventListener('click', () => {
        count--;
        countEl.textContent = count;
    });
}
```

## 5. 로컬 개발 서버 실행

### 정적 서버 사용

**package.json 생성:**

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "scripts": {
    "dev": "npx serve .",
    "dev:watch": "npx live-server --port=8080"
  },
  "devDependencies": {
    "live-server": "^1.2.2"
  }
}
```

### Claude에게 요청

```
개발 서버를 시작해줘.

npm install
npm run dev:watch

브라우저에서 http://localhost:8080 확인.
```

## 6. Cloudflare Pages 배포

### Git 저장소 연결

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/myapp.git
git push -u origin main
```

### Cloudflare Pages 설정

1. **Cloudflare 대시보드 접속:**
   - https://dash.cloudflare.com/

2. **Pages → Create a project**

3. **Git 저장소 연결:**
   - GitHub 저장소 선택

4. **빌드 설정:**
   ```
   Framework preset: None
   Build command: (비워둠)
   Build output directory: /
   ```

5. **배포 시작:**
   - "Save and Deploy" 클릭

### 배포 완료

```
https://myapp.pages.dev
```

## 7. 실전 프롬프트 예시

### 블로그 페이지 추가

```
블로그 목록 페이지를 추가해줘.

경로: /blog
뷰: src/views/blog.html
로직: src/logic/blog.js

기능:
- Mock API에서 글 목록 가져오기
- 카드 형태로 표시 (제목, 요약, 날짜)
- 상세 페이지로 링크

router.js에 라우트 추가.
```

### 연락처 폼 추가

```
연락처 폼 페이지를 만들어줘.

경로: /contact
필드: name, email, message

기능:
- 폼 유효성 검사
- Cloudflare Workers 백엔드로 전송 (/api/contact)
- 성공 시 감사 메시지

AJAX로 처리.
```

### 404 페이지 추가

```
404 에러 페이지를 만들어줘.

경로: /404
뷰: src/views/404.html

내용:
- "페이지를 찾을 수 없습니다" 메시지
- 홈으로 돌아가기 버튼
- 검색 기능 추가

router.js에서 매칭 안 되면 /404로 리다이렉트.
```

## 8. 다음 단계

프로젝트 기본 구조가 완성되었습니다. 이제 다음 가이드를 참고하여 기능을 확장하세요:

- [Pages 프로젝트 구조](pages-structure.md)
- [라우팅 및 페이지 개발](pages-routing.md)
- [Functions 개발](pages-functions.md)
- [빌드 및 배포](pages-deployment.md)

## 체크리스트

프로젝트 시작 시 확인사항:

- [ ] index.html 설정 완료
- [ ] router.js 구현 완료
- [ ] 첫 페이지 작동 확인
- [ ] 로컬 개발 서버 실행 확인
- [ ] Git 저장소 초기화 완료
- [ ] Cloudflare Pages 배포 완료
- [ ] 배포된 URL 접속 확인

## 문제 해결

### 라우터가 작동하지 않음

**증상:** 링크 클릭 시 페이지 새로고침

**해결:**
```
data-link 속성이 있는지 확인해줘.
router.js가 제대로 로드되는지 체크.
```

### 뷰가 로드되지 않음

**증상:** 빈 화면 또는 404 오류

**해결:**
```
뷰 파일 경로가 올바른지 확인해줘.
개발 서버가 루트 디렉토리에서 실행 중인지 체크.
```

### CORS 오류

**증상:** `Access-Control-Allow-Origin` 오류

**해결:**
```
로컬 개발 시 file:// 프로토콜 사용 금지.
반드시 HTTP 서버 사용 (npm run dev:watch).
```

## 관련 문서

- [.claude 설정 예제](pages-claude-setup.md)
- [프로젝트 구조 표준](project-structure.md)
- [코딩 규칙](coding-rules.md)
