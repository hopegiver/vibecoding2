# Vue Zero - 시작하기

Vue Zero + Cloudflare Workers 기반 풀스택 웹 애플리케이션을 시작하는 방법을 안내합니다.

## 왜 Vue Zero인가?

일반적인 프론트엔드 프레임워크(React, Next.js, Nuxt 등)로 바이브코딩을 하면 다음과 같은 문제가 생깁니다:

- **빌드 에러의 늪**: AI가 코드를 생성해도 Webpack/Vite 빌드에서 실패하면 사용자가 직접 해결해야 합니다
- **설정 지옥**: `tsconfig.json`, `vite.config.ts`, `.eslintrc` 등 설정 파일 간 충돌을 AI가 완벽히 해결하기 어렵습니다
- **프레임워크 마법**: `<script setup>`, auto-import, 컴파일러 매크로 등 프레임워크 고유 문법은 AI가 실수할 가능성이 높습니다

Vue Zero는 이런 문제를 원천적으로 제거한 **AI 바이브코딩 전용 프레임워크**입니다.

### AI 바이브코딩에 최적화된 이유

| 기존 프레임워크 | Vue Zero |
|----------------|----------|
| 빌드 필수 (Webpack/Vite) | **빌드 없음** — CDN에서 브라우저로 직접 실행 |
| 빌드 에러 시 사용자가 해결 | **빌드가 없으니 빌드 에러도 없음** |
| TypeScript, JSX 등 다양한 문법 | **Options API 하나만 사용** — AI 생성 코드의 일관성 보장 |
| 복잡한 설정 파일 다수 | **설정 파일 최소화** — `wrangler.toml` 하나로 충분 |
| 프레임워크 고유 문법 학습 필요 | **표준 Vue 3 문법만 사용** — 모든 AI가 이미 알고 있는 지식 |
| 프론트/백엔드 별도 구성 | **풀스택 일체형** — 하나의 프로젝트에서 화면과 API 모두 처리 |

### 핵심 특징

- **빌드 불필요**: CDN을 통해 브라우저에서 바로 실행. AI가 파일을 저장하면 즉시 결과 확인
- **파일 기반 라우팅**: `.vue` 파일을 만들면 자동으로 URL 경로 생성. 설정 파일 수정 불필요
- **자동 등록**: Hook이 페이지, 컴포넌트, API 라우트를 자동으로 등록. AI가 파일만 만들면 끝
- **풀스택 일체형**: Cloudflare Workers (Hono) 백엔드와 한 프로젝트에서 통합
- **디자인 시스템 내장**: Bootstrap 5 + 디자인 토큰이 포함되어 AI가 바로 예쁜 UI 생성 가능

## 전제조건

- Node.js 18 이상
- Cloudflare 계정
- VSCode + Claude Code

## 1단계: 템플릿 클론

```bash
git clone https://github.com/hopegiver/vue-zero-template my-app
cd my-app

# 기존 git 히스토리 제거 후 새로 초기화
rm -rf .git
git init

# 의존성 설치
npm install
```

## 2단계: 로컬 서버 실행

```bash
wrangler dev
```

`http://localhost:8787`에서 앱을 확인할 수 있습니다.

## 3단계: AI에게 요청하기

프로젝트가 실행되면 Claude Code에게 자연어로 요청합니다:

```
"사용자 목록 페이지를 만들어줘. /api/users에서 데이터를 가져와서 테이블로 보여줘."
```

```
"로그인 페이지를 추가하고, 인증되지 않은 사용자는 접근할 수 없게 해줘."
```

```
"갤러리 페이지를 추가해줘. 이미지를 그리드로 보여주고 클릭하면 크게 볼 수 있게."
```

AI가 `app/pages/`에 프론트엔드 페이지를, `server/api/`와 `server/dao/`에 백엔드 API를 자동으로 생성합니다.

## 알아두면 좋은 핵심 개념

### 파일 = URL

`app/pages/` 폴더에 파일을 만들면 자동으로 URL이 됩니다:

| 파일 | URL |
|------|-----|
| `pages/index.vue` | `/` |
| `pages/about.vue` | `/about` |
| `pages/users/index.vue` | `/users` |
| `pages/users/[id].vue` | `/users/:id` (동적) |

### 프론트엔드 + 백엔드 분리

| 폴더 | 역할 |
|------|------|
| `app/` | 프론트엔드 — 사용자에게 보이는 화면 |
| `server/` | 백엔드 — API, 데이터베이스 처리 |

### 자동 등록 (Hook)

AI가 `.vue` 파일이나 API 파일을 생성하면 **Hook이 자동으로 등록 파일을 갱신**합니다. 직접 파일을 추가한 경우에만 수동 실행이 필요합니다:

```bash
npm run scan    # pages.json, components.json, _registry.js 갱신
```

## 주요 명령어

| 명령어 | 설명 |
|--------|------|
| `wrangler dev` | 로컬 개발 서버 실행 |
| `wrangler deploy` | 프로덕션 배포 |
| `npm run scan` | 페이지/컴포넌트 등록 목록 자동 갱신 |
| `npm run test` | E2E 테스트 실행 |

## 체크리스트

- [ ] vue-zero-template 클론 완료
- [ ] `npm install` 완료
- [ ] 로컬 서버 작동 확인 (`wrangler dev`)
- [ ] 첫 페이지 추가 요청 (AI에게)
- [ ] `npm run scan` 실행

## 문제 해결

### 페이지가 표시되지 않음

- `npm run scan` 실행하여 `pages.json` 갱신
- 브라우저 콘솔에서 에러 확인

### 로컬 서버 실행 실패

- `npm install` 실행 여부 확인
- `wrangler.toml`의 `main` 경로 확인

### API 호출이 404 반환

- `wrangler.toml`의 `run_worker_first = ["/api/*"]` 설정 확인
- API 경로가 `/api/`로 시작하는지 확인

## 관련 문서

- [프로젝트 구조](workers-structure.md)
- [배포](workers-deployment.md)

[← 목차로 돌아가기](../_sidebar.md)
