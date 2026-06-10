# Vue Zero - 프로젝트 구조

프로젝트의 폴더 구조와 각 폴더의 역할을 안내합니다. AI에게 기능을 요청할 때 어떤 폴더가 어떤 역할을 하는지 이해하면 더 정확한 요청을 할 수 있습니다.

## 전체 구조

```
my-app/
├── app/                        # 프론트엔드 (화면)
│   ├── index.html              # 앱 진입점
│   ├── pages/                  # 페이지 (파일 = URL)
│   │   ├── pages.json          # 페이지 등록 목록
│   │   ├── index.vue           # /
│   │   ├── about.vue           # /about
│   │   ├── login.vue           # /login
│   │   ├── 404.vue             # 404 페이지
│   │   ├── users/
│   │   │   ├── index.vue       # /users
│   │   │   └── [id].vue        # /users/:id
│   │   └── admin/
│   │       ├── index.vue       # /admin
│   │       └── members.vue     # /admin/members
│   ├── components/             # 재사용 컴포넌트
│   │   ├── components.json     # 컴포넌트 등록 목록
│   │   ├── AppButton.vue
│   │   └── UserCard.vue
│   ├── layouts/                # 페이지 레이아웃
│   │   ├── default.vue         # 기본 레이아웃
│   │   └── admin.vue           # 관리자 레이아웃
│   └── assets/
│       └── css/
│           └── base.css        # 디자인 토큰
├── server/                     # 백엔드 (API)
│   ├── index.js                # 앱 초기화
│   ├── api/                    # API 엔드포인트
│   │   ├── auth.js
│   │   ├── users.js
│   │   ├── members.js
│   │   ├── gallery.js
│   │   └── stats.js
│   ├── dao/                    # 데이터베이스 처리
│   │   ├── users.js
│   │   ├── members.js
│   │   ├── gallery.js
│   │   └── stats.js
│   ├── middleware/             # 인증 등 공통 처리
│   │   └── auth.js
│   └── utils/                  # 유틸리티
│       └── response.js
├── bin/                        # CLI 도구
│   ├── scan.js                 # npm run scan
│   └── hook-scan.js            # git hook용
├── tests/                      # E2E 테스트
│   └── e2e.spec.js
├── CLAUDE.md                   # AI 개발 가이드
├── wrangler.toml               # Cloudflare 설정
└── package.json
```

## app/ — 프론트엔드 (화면)

사용자에게 보이는 모든 화면입니다. 빌드 없이 브라우저에서 직접 실행됩니다.

### pages/ — 페이지

**가장 자주 다루는 폴더입니다.** 여기에 `.vue` 파일을 만들면 자동으로 URL이 됩니다.

| 파일 패턴 | 의미 | 예시 |
|-----------|------|------|
| `이름.vue` | 정적 페이지 | `about.vue` → `/about` |
| `폴더/index.vue` | 폴더의 메인 페이지 | `users/index.vue` → `/users` |
| `[파라미터].vue` | 동적 페이지 | `users/[id].vue` → `/users/123` |
| `404.vue` | 없는 페이지 접속 시 | 자동 감지 (등록 불필요) |

> 페이지를 추가/삭제한 후 `npm run scan`을 실행하여 `pages.json`을 갱신하세요.

### components/ — 재사용 컴포넌트

여러 페이지에서 공통으로 사용하는 UI 조각입니다. `components.json`에 등록하면 모든 페이지에서 바로 사용할 수 있습니다.

> AI에게 "공통 버튼 컴포넌트 만들어줘"라고 요청하면 여기에 생성됩니다.

### layouts/ — 레이아웃

페이지를 감싸는 틀입니다. 네비게이션 바, 사이드바, 푸터 같은 공통 UI를 담습니다.

| 파일 | 용도 |
|------|------|
| `default.vue` | 일반 페이지용 (자동 적용) |
| `admin.vue` | 관리자 페이지용 (사이드바 등) |

> AI에게 "관리자 레이아웃에 사이드바 메뉴 추가해줘"라고 요청할 수 있습니다.

### assets/css/ — 스타일

`base.css`에 Bootstrap 5를 확장한 디자인 토큰(색상, 간격, 그림자 등)이 정의되어 있습니다.

## server/ — 백엔드 (API)

Hono 프레임워크 기반 API 서버입니다. Cloudflare Workers 위에서 실행됩니다.

### api/ — API 엔드포인트

프론트엔드에서 호출하는 API입니다. `/api/users`, `/api/auth` 같은 경로를 처리합니다.

> AI에게 "게시물 CRUD API 만들어줘"라고 요청하면 여기에 생성됩니다.

### dao/ — 데이터베이스 처리

실제 데이터베이스(D1, KV)에 접근하는 로직입니다. API 파일과 1:1로 대응됩니다.

| api/ | dao/ | 역할 |
|------|------|------|
| `users.js` | `users.js` | 사용자 데이터 |
| `gallery.js` | `gallery.js` | 갤러리 데이터 |

### middleware/ — 공통 처리

모든 API 요청에 공통으로 적용되는 처리입니다. JWT 인증이 대표적입니다.

### utils/ — 유틸리티

API 응답 포맷 등 공통 헬퍼 함수입니다.

## 자동화: Hook + Scan

vue-zero-template에는 **AI가 파일을 추가/수정할 때 자동으로 등록 파일을 갱신하는 Hook**이 설정되어 있습니다.

### 동작 방식

`.claude/settings.json`에 다음 Hook이 설정되어 있습니다:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node bin/hook-scan.js || true"
          }
        ]
      }
    ]
  }
}
```

AI가 파일을 생성하거나 수정(Write/Edit)할 때마다 `bin/hook-scan.js`가 자동 실행됩니다:

| AI가 수정한 파일 | 자동 실행 내용 |
|-----------------|---------------|
| `.vue` 파일 | `pages.json`, `components.json` 자동 갱신 |
| `server/api/*.js` 파일 | `server/_registry.js` 자동 갱신 (API 라우트 등록) |

### 사용자가 알아야 할 점

- **`npm run scan`을 수동으로 실행할 필요가 거의 없습니다.** AI가 코딩할 때 Hook이 자동으로 처리합니다.
- 직접 파일을 추가/삭제한 경우에만 `npm run scan`을 수동 실행하세요.
- `server/_registry.js`는 자동 생성 파일입니다. 직접 수정하지 마세요.

## 설정 파일

### wrangler.toml

Cloudflare Workers의 핵심 설정입니다. 프로젝트명, 백엔드 진입점, 정적 파일 경로를 지정합니다.

| 설정 | 의미 |
|------|------|
| `name` | 프로젝트명 (배포 시 사용) |
| `main` | 백엔드 진입점 (`server/index.js`) |
| `[assets] directory` | 프론트엔드 폴더 (`./app`) |
| `run_worker_first` | API 경로만 백엔드가 처리 (`["/api/*"]`) |

### CLAUDE.md

AI가 자동으로 참조하는 프로젝트 가이드입니다. 코딩 규칙, 폴더 구조, 주요 명령어가 담겨 있어 AI가 프로젝트에 맞는 코드를 생성합니다.

> 이 파일을 수정하면 AI의 코딩 방식을 바꿀 수 있습니다.

## AI에게 요청할 때 팁

기능을 요청할 때 관련 폴더를 언급하면 더 정확한 결과를 얻을 수 있습니다:

| 요청 예시 | AI가 작업하는 폴더 |
|-----------|-------------------|
| "사용자 목록 페이지 만들어줘" | `pages/`, `api/`, `dao/` |
| "네비게이션에 메뉴 추가해줘" | `layouts/` |
| "별점 컴포넌트 만들어줘" | `components/` |
| "D1 데이터베이스 연결해줘" | `wrangler.toml`, `dao/` |
| "로그인 기능 추가해줘" | `pages/`, `api/`, `middleware/` |

## 관련 문서

- [시작하기](workers-getting-started.md)
- [배포](workers-deployment.md)

[← 목차로 돌아가기](../_sidebar.md)
