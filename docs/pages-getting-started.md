# Cloudflare Pages - 프로젝트 시작하기

Cloudflare Pages + ViewLogic Router 기반 SPA 프로젝트를 시작하는 방법을 안내합니다.

## 전제조건

- Git
- Cloudflare 계정
- VSCode + Claude Code

## 1. viewlogic-template으로 시작하기

### 템플릿 클론

```bash
# viewlogic-template 클론
git clone https://github.com/hopegiver/viewlogic-template myapp
cd myapp

# 기존 git 히스토리 제거 후 새로 초기화
rm -rf .git
git init
```

### 템플릿 기본 구조

```
myapp/
├── CLAUDE.md                      # AI 개발 가이드 (핵심 규칙)
├── .claude/
│   ├── rules/                     # 자동 적용 규칙
│   │   ├── viewlogic-guide.md     # ViewLogic 개발 규칙
│   │   └── style-guide.md         # CSS 스타일 규칙
│   ├── commands/                  # 슬래시 커맨드
│   │   ├── create-page.md         # /create-page
│   │   ├── create-component.md    # /create-component
│   │   └── create-layout.md       # /create-layout
│   └── templates/                 # 코드 생성 템플릿
│       ├── page.md                # 페이지 (정적/목록/상세/폼)
│       ├── component.md           # 컴포넌트 (기본/슬롯/v-model)
│       └── layout.md              # 레이아웃 (네비/사이드바)
├── docs/                          # 상세 개발 문서
│   ├── routing.md                 # 라우팅, 페이지 이동
│   ├── data-fetching.md           # dataURL, API 호출
│   ├── forms.md                   # 폼 처리
│   ├── api.md                     # $api 메서드
│   ├── auth.md                    # 인증
│   ├── i18n.md                    # 다국어
│   ├── components.md              # 컴포넌트 생성/등록
│   ├── components-builtin.md      # 내장 컴포넌트
│   ├── layout.md                  # 레이아웃 시스템
│   ├── patterns.md                # 공통 패턴
│   ├── advanced.md                # 고급 기능
│   └── configuration.md           # 설정 옵션
├── index.html                     # 엔트리 포인트
├── css/
│   └── base.css                   # 커스텀 스타일 (Bootstrap 보완)
├── src/
│   ├── views/                     # HTML 뷰
│   │   ├── layout/
│   │   │   └── default.html       # 공통 레이아웃 (헤더/푸터)
│   │   ├── home.html
│   │   ├── about.html
│   │   ├── contact.html
│   │   ├── 404.html
│   │   └── error.html
│   ├── logic/                     # JavaScript 로직
│   │   ├── layout/
│   │   │   └── default.js
│   │   ├── home.js
│   │   ├── about.js
│   │   ├── contact.js
│   │   ├── 404.js
│   │   └── error.js
│   └── components/                # 재사용 컴포넌트
│       ├── Table.js
│       ├── Sidebar.js
│       ├── Loading.js
│       ├── FileUpload.js
│       ├── DatePicker.js
│       ├── HtmlInclude.js
│       └── DynamicInclude.js
└── README.md
```

### CLAUDE.md

Claude Code가 자동으로 참조하는 프로젝트 가이드입니다:
- 프로젝트 구조 및 기술 스택
- 핵심 규칙 6가지 (파일 쌍, 폴더=라우트, CSS 금지, navigateTo 등)
- 슬래시 커맨드 (`/create-page`, `/create-component`, `/create-layout`)
- 템플릿 및 상세 문서 안내

### .claude/ 폴더

**rules/** - 자동 적용 규칙:
- `viewlogic-guide.md`: 파일 구조, 필수 패턴, 내장 메서드, 금지 사항, 참조 트리거
- `style-guide.md`: Bootstrap 5 최대 활용, CSS 변수 사용, 반응형 규칙

**commands/** - 슬래시 커맨드:
- `/create-page` - 페이지(view + logic) 생성
- `/create-component` - 재사용 컴포넌트 생성
- `/create-layout` - 레이아웃 생성

**templates/** - 코드 생성 시 참조하는 표준 템플릿:
- `page.md`: 4가지 변형 (정적, 목록, 상세, 폼)
- `component.md`: 3가지 변형 (기본, 슬롯, v-model)
- `layout.md`: 2가지 변형 (네비게이션, 사이드바)

### docs/ (상세 개발 문서)

| 문서 | 내용 |
|------|------|
| routing.md | 파일 기반 라우팅, 페이지 이동, 파라미터 |
| data-fetching.md | dataURL 자동 로딩, 수동 API 호출 |
| forms.md | 명령형/선언적 폼 처리 |
| api.md | $api 메서드 (GET/POST/PUT/DELETE) |
| auth.md | 인증 설정, 로그인/로그아웃, 토큰 관리 |
| i18n.md | 다국어 설정, 언어 전환 |
| components.md | 컴포넌트 생성/등록 |
| components-builtin.md | 내장 컴포넌트 (Table, Sidebar 등) |
| layout.md | 레이아웃 시스템 |
| patterns.md | 공통 패턴 (로딩, 에러, 밸리데이션) |
| advanced.md | 라이프사이클, computed, watch, 캐싱 |
| configuration.md | ViewLogicRouter 전체 설정 옵션 |

## 2. index.html

템플릿의 index.html:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My App</title>
    <!-- Bootstrap 5 CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="css/base.css">
</head>
<body>
    <div id="app"></div>

    <!-- Vue 3 -->
    <script src="https://cdn.jsdelivr.net/npm/vue@3/dist/vue.global.prod.js"></script>
    <!-- ViewLogic Router -->
    <script src="https://cdn.jsdelivr.net/npm/viewlogic@1.4.0/dist/viewlogic-router.min.js"></script>
    <!-- Bootstrap 5 JS -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>

    <script>
        const router = new ViewLogicRouter({
            environment: 'development'
        });
    </script>
</body>
</html>
```

**핵심 요소:**
- `<div id="app">`: ViewLogic이 뷰를 렌더링하는 위치
- Bootstrap 5.3.3: 레이아웃/UI 스타일
- Vue 3 (prod): 데이터 바인딩, 이벤트 처리
- ViewLogic 1.4.0: 파일 기반 SPA 라우팅
- `environment: 'development'`: 개발 모드 (디버그 로깅)

## 3. 페이지 개발

레이아웃, 페이지 추가, API 연동 등 ViewLogic 코드 작성법은 [ViewLogic 사용법](pages-viewlogic.md) 참고.

## 4. 로컬 개발 서버 실행

빌드 없이 정적 서버로 바로 실행:

```bash
# Python
python -m http.server 8000

# 또는 Node.js
npx serve .
```

브라우저에서 `http://localhost:8000` 접속.

## 5. Cloudflare Pages 배포

### Git 저장소 연결

```bash
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/myapp.git
git push -u origin main
```

### Cloudflare Pages 설정

1. https://dash.cloudflare.com/ → Pages → Create a project
2. GitHub 저장소 연결
3. 빌드 설정:
   ```
   Framework preset: None
   Build command: (비워둠)
   Build output directory: /
   ```
4. "Save and Deploy" 클릭

배포 완료: `https://myapp.pages.dev`

## 6. 프로젝트 커스터마이즈

### 레이아웃 수정

`src/views/layout/default.html`의 네비게이션, 푸터를 프로젝트에 맞게 수정.

### 기존 페이지 수정

템플릿에 포함된 home, about, contact 페이지를 프로젝트 내용으로 변경.

### .claude/ 커스터마이즈

프로젝트에 맞는 규칙 추가:

```
.claude/rules/
├── viewlogic-guide.md     # ViewLogic 규칙 (템플릿 제공)
├── style-guide.md         # CSS 규칙 (템플릿 제공)
└── project-rules.md       # 프로젝트별 규칙 (직접 추가)
```

## 체크리스트

프로젝트 시작 시 확인사항:

- [ ] viewlogic-template 클론 완료
- [ ] CLAUDE.md 및 .claude/ 확인
- [ ] index.html 타이틀 변경
- [ ] layout/default.html 네비게이션 수정
- [ ] 첫 페이지 작동 확인 (로컬 서버)
- [ ] Git 저장소 초기화 완료
- [ ] Cloudflare Pages 배포 완료

## 문제 해결

### 페이지가 표시되지 않음

**증상:** 빈 화면

**확인:**
- `<div id="app"></div>` 존재 여부
- 브라우저 콘솔 에러 메시지
- views/logic 파일 경로 일치 여부 (대소문자 구분)

### CORS 오류

**증상:** `Access-Control-Allow-Origin` 오류

**해결:**
- `file://` 프로토콜 사용 금지
- 반드시 HTTP 서버 사용 (`python -m http.server`)

### 라우트 인식 안 됨

**확인:**
- URL에 `#/` 포함 여부 (예: `http://localhost:8000/#/home`)
- views와 logic 파일명 동일 여부

## 관련 문서

- [프로젝트 구조](pages-structure.md)
- [ViewLogic 사용법](pages-viewlogic.md)
- [빌드 및 배포](pages-deployment.md)
- [프로젝트 구조 표준](project-structure.md)
