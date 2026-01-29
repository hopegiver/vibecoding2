# Cloudflare Workers - 프로젝트 시작하기

Cloudflare Workers로 서버리스 API 프로젝트를 시작하는 방법을 안내합니다.

## 전제조건

- Node.js 18 이상
- npm 또는 yarn
- Cloudflare 계정
- VSCode + Claude Code

## 1. 프로젝트 초기 설정

### C3 CLI로 생성

```bash
# C3 (create-cloudflare-cli) 사용
npm create cloudflare@latest myworker

# 프롬프트에서 선택:
# - Type: Web API
# - Template: Hello World
# - TypeScript: Yes
# - Git: Yes
# - Deploy: No (나중에)
```

### Claude에게 요청하기

```
Cloudflare Workers 프로젝트를 새로 만들어줘.
프로젝트 이름: myworker
템플릿: RESTful API

구조:
- src/index.ts (메인 파일)
- wrangler.toml (설정)
- TypeScript 사용
```

## 2. 프로젝트 구조

### 기본 구조

```
myworker/
├── src/
│   ├── index.ts               # 메인 Worker
│   ├── router.ts              # 라우터
│   └── handlers/              # 핸들러
│       ├── users.ts
│       └── posts.ts
├── wrangler.toml              # 설정 파일
├── package.json
├── tsconfig.json
└── .gitignore
```

## 3. 첫 Worker 만들기

### src/index.ts

```typescript
export interface Env {
    // 환경 변수와 바인딩
}

export default {
    async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
        const url = new URL(request.url);

        if (url.pathname === '/') {
            return new Response(JSON.stringify({
                message: 'Hello from Cloudflare Workers!'
            }), {
                headers: { 'Content-Type': 'application/json' }
            });
        }

        if (url.pathname === '/api/hello') {
            const name = url.searchParams.get('name') || 'World';
            return new Response(JSON.stringify({
                message: `Hello, ${name}!`
            }), {
                headers: { 'Content-Type': 'application/json' }
            });
        }

        return new Response('Not Found', { status: 404 });
    }
};
```

**핵심 개념:**
- `fetch`: 모든 HTTP 요청의 진입점
- `Request`: 들어오는 요청
- `Response`: 응답
- `Env`: 환경 변수 및 바인딩

## 4. wrangler.toml 설정

```toml
name = "myworker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# 환경 변수
[vars]
ENVIRONMENT = "development"

# D1 데이터베이스
[[d1_databases]]
binding = "DB"
database_name = "myworker-db"
database_id = "xxx-xxx-xxx"

# KV 네임스페이스
[[kv_namespaces]]
binding = "CACHE"
id = "xxx-xxx-xxx"

# R2 버킷
[[r2_buckets]]
binding = "UPLOADS"
bucket_name = "myworker-uploads"
```

## 5. 로컬 개발 서버 실행

### 개발 서버 시작

```bash
# 로컬 실행
npm run dev

# 또는
npx wrangler dev

# 특정 포트
npx wrangler dev --port 8787
```

**접속:**
```
http://localhost:8787/
http://localhost:8787/api/hello?name=Claude
```

## 6. 라우터 구현

### src/router.ts

```typescript
type Handler = (request: Request, env: any) => Promise<Response>;

interface Route {
    method: string;
    pattern: RegExp;
    handler: Handler;
}

export class Router {
    private routes: Route[] = [];

    get(path: string, handler: Handler) {
        this.addRoute('GET', path, handler);
    }

    post(path: string, handler: Handler) {
        this.addRoute('POST', path, handler);
    }

    put(path: string, handler: Handler) {
        this.addRoute('PUT', path, handler);
    }

    delete(path: string, handler: Handler) {
        this.addRoute('DELETE', path, handler);
    }

    private addRoute(method: string, path: string, handler: Handler) {
        // 경로를 정규식으로 변환
        const pattern = new RegExp('^' + path.replace(/:\w+/g, '([^/]+)') + '$');
        this.routes.push({ method, pattern, handler });
    }

    async handle(request: Request, env: any): Promise<Response> {
        const url = new URL(request.url);
        const method = request.method;

        for (const route of this.routes) {
            if (route.method === method) {
                const match = url.pathname.match(route.pattern);
                if (match) {
                    // 파라미터 추출
                    request.params = match.slice(1);
                    return route.handler(request, env);
                }
            }
        }

        return new Response('Not Found', { status: 404 });
    }
}
```

### src/index.ts (라우터 사용)

```typescript
import { Router } from './router';

const router = new Router();

router.get('/', async (request, env) => {
    return new Response(JSON.stringify({
        message: 'API is running'
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
});

router.get('/api/users', async (request, env) => {
    // 사용자 목록
    return new Response(JSON.stringify([
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' }
    ]), {
        headers: { 'Content-Type': 'application/json' }
    });
});

router.get('/api/users/:id', async (request, env) => {
    const id = request.params[0];
    return new Response(JSON.stringify({
        id,
        name: 'User ' + id
    }), {
        headers: { 'Content-Type': 'application/json' }
    });
});

export default {
    async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
        return router.handle(request, env);
    }
};
```

## 7. 배포

### Wrangler로 배포

```bash
# 프로덕션 배포
npx wrangler deploy

# 배포 후 URL
# https://myworker.<subdomain>.workers.dev
```

### package.json 스크립트

```json
{
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "wrangler deploy",
    "tail": "wrangler tail"
  }
}
```

## 8. 실전 프롬프트 예시

### CRUD API 추가

```
사용자 CRUD API를 만들어줘.

엔드포인트:
- GET /api/users - 목록
- GET /api/users/:id - 상세
- POST /api/users - 생성
- PUT /api/users/:id - 수정
- DELETE /api/users/:id - 삭제

D1 데이터베이스 사용.
router.ts에 라우트 추가.
src/handlers/users.ts에 핸들러 작성.
```

### 인증 미들웨어 추가

```
JWT 인증 미들웨어를 추가해줘.

기능:
- Authorization 헤더 확인
- JWT 토큰 검증
- 토큰에서 사용자 정보 추출
- request에 user 정보 추가

특정 라우트에만 적용 (/api/users/* 등).
```

### CORS 설정 추가

```
CORS 설정을 추가해줘.

허용:
- Origin: * (또는 특정 도메인)
- Methods: GET, POST, PUT, DELETE
- Headers: Content-Type, Authorization

OPTIONS 요청 처리.
모든 응답에 CORS 헤더 추가.
```

## 9. 다음 단계

프로젝트 기본 구조가 완성되었습니다. 이제 다음 가이드를 참고하여 기능을 확장하세요:

- [라우팅 및 요청 처리](workers-routing.md)
- [API 개발](workers-api.md)
- [D1 데이터베이스 활용](workers-d1.md)
- [KV 스토리지 활용](workers-kv.md)

## 체크리스트

프로젝트 시작 시 확인사항:

- [ ] Wrangler CLI가 설치되어 있는가?
- [ ] wrangler.toml 설정이 완료되었는가?
- [ ] TypeScript 설정이 올바른가?
- [ ] 로컬 개발 서버가 작동하는가?
- [ ] 라우터가 구현되어 있는가?
- [ ] 배포가 성공했는가?
- [ ] 배포된 URL에 접속할 수 있는가?

## 문제 해결

### Wrangler 로그인 실패

**증상:** `wrangler login` 오류

**해결:**
```
브라우저에서 Cloudflare에 로그인.
wrangler login --scopes-list로 권한 확인.
```

### 로컬 서버 실행 실패

**증상:** `Error: Could not resolve "src/index.ts"`

**해결:**
```
wrangler.toml의 main 경로 확인.
TypeScript가 설치되어 있는지 확인 (npm install -D typescript).
```

### 배포 실패

**증상:** `Error: Authentication error`

**해결:**
```
wrangler login 재실행.
Cloudflare 계정 권한 확인.
```

## 관련 문서

- [.claude 설정 예제](workers-claude-setup.md)
- [프로젝트 구조 표준](project-structure.md)
- [코딩 규칙](coding-rules.md)
