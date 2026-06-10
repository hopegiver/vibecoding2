# CLAUDE.md 작성 가이드

## 개요

`CLAUDE.md`는 프로젝트 루트에 위치하는 파일로, Claude Code에게 **프로젝트 전체의 컨텍스트**를 제공합니다. 이 파일은 프로젝트가 무엇을 하는지, 어떤 기술을 사용하는지, 어떻게 구성되어 있는지를 명확하게 설명하여 Claude Code가 프로젝트를 빠르게 이해할 수 있도록 도와줍니다.

## 왜 필요한가?

Claude Code는 대화를 시작할 때마다 **컨텍스트를 새로 구축**합니다. CLAUDE.md가 있으면:

- 🚀 프로젝트 파악 시간 단축 (매번 설명할 필요 없음)
- 📋 일관된 컨텍스트 유지 (누가 사용해도 동일한 이해)
- 🎯 정확한 코드 생성 (프로젝트 구조와 패턴 이해)
- 🔄 작업 재개 용이 (이전 작업 내용 기록)

## 적정 크기 가이드

### 권장 크기
- **CLAUDE.md 파일**: 3,000-5,000 토큰 (약 9,000-15,000자)

### 토큰 수 확인
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer) 사용
- 한글 기준: 글자 수 × 3 ≈ 토큰 수

### 크기 초과 시
CLAUDE.md가 너무 길면:
- 중요한 정보를 압축
- 상세한 내용은 별도 문서로 분리하고 링크만 제공
- 500줄 이하로 유지 권장

### 크기 관리 팁
- 핵심 정보만 포함 (개요, 구조, 핵심 규칙)
- 상세 규칙은 `.claude/rules/`로
- 코드 예제는 `.claude/templates/`로
- 작업 이력은 최근 3-5개만 유지

## 기본 구조

```markdown
# 프로젝트명

## 프로젝트 개요
- 프로젝트가 무엇을 하는지
- 주요 기능
- 기술 스택

## 핵심 아키텍처
- 프로젝트 구조
- 주요 패턴
- 데이터 흐름

## 주요 페이지/기능 설명
- 핵심 기능별 상세 설명

## 개발 규칙
- 코딩 원칙
- 금지 사항
- 권장 패턴

## 현재 작업 상태 (선택)
- 최근 작업 내용
- 다음 할 일
```

## 작성 원칙

### 1. 명확한 프로젝트 개요로 시작

첫 문단에서 프로젝트가 무엇인지 명확하게 설명하세요.

```markdown
# 성과운영 시스템 (GPM - Goal Performance Management)

## 프로젝트 개요
목표 중심의 성과 관리 및 개인 성장 플랫폼. OKR/MBO 기반 목표 설정, 실행 관리, 성장 추적, 평가를 통합 지원.

**기술 스택:** Vue 3 + ViewLogic Router + Bootstrap 5 + Chart.js
```

### 2. 아키텍처 패턴을 시각적으로 표현

코드 블록과 트리 구조를 활용하세요.

```markdown
## 핵심 아키텍처

### 1. ViewLogic Router 패턴
\`\`\`
src/
├── views/         # HTML 템플릿 (CSS 금지)
│   ├── goals/my-goals.html
│   └── team/tasks.html
└── logic/         # JavaScript 로직
    ├── goals/my-goals.js
    └── team/tasks.js
\`\`\`

- **파일명 = 라우트**: `goals/my-goals.html` → `#/goals/my-goals`
- **분리 원칙**: HTML과 JS 완전 분리
```

### 3. 핵심 규칙을 명시

Claude Code가 반드시 지켜야 할 규칙을 강조하세요.

```markdown
## CSS 규칙 (.claude/rules/style-guide.md)

### 절대 원칙
❌ **HTML 파일에 `<style>` 태그 절대 금지**
✅ 모든 CSS는 `css/base.css`에 작성

### Bootstrap 우선
\`\`\`html
<!-- ✅ 올바름 -->
<div class="d-flex gap-3 mb-4">

<!-- ❌ 금지 -->
<div style="display: flex; gap: 12px;">
\`\`\`
```

### 4. 주요 데이터 구조 포함

API 응답이나 주요 객체 구조를 명시하세요.

```markdown
## 주요 데이터 흐름

### 목표 데이터 구조
\`\`\`javascript
{
  id: 1,
  companyKPIId: 1,          // 회사 목표 연계 (필수)
  category: "재무",          // BSC 관점
  title: "신규 프로덕트 출시 3개 이상",
  targetValue: 3,           // 목표 수치
  currentValue: 2,          // 현재 수치
  achievement: 70,          // 달성률 (%)
  status: "진행중"          // 진행중|완료|지연|보류
}
\`\`\`
```

### 5. 작업 상태 업데이트 (선택)

장기 프로젝트는 현재 상태를 기록하면 유용합니다.

```markdown
## 다음 개발 예정

1. **실제 API 연동**
   - Mock JSON → REST API 전환
   - `this.$api.get/post/put/delete` 활용

2. **개인 목표 관리**
   - 나의 목표 페이지 완성
   - 목표 상세 페이지 (`/goals/detail/:id`)

3. **실행 관리 강화**
   - 오늘의 업무 페이지
   - 이번 주 페이지

---
**마지막 업데이트:** 2024-01-19
**개발 규칙:** `.claude/rules/` 폴더 참조
```

## 실전 예제

### 예제 1: Pages 프로젝트

**파일: `CLAUDE.md`**

```markdown
# 성과운영 시스템 (GPM - Goal Performance Management)

## 프로젝트 개요
목표 중심의 성과 관리 및 개인 성장 플랫폼. OKR/MBO 기반 목표 설정, 실행 관리, 성장 추적, 평가를 통합 지원.

**기술 스택:** Vue 3 + ViewLogic Router + Bootstrap 5 + Chart.js

## 핵심 아키텍처

### 1. ViewLogic Router 패턴
\`\`\`
src/
├── views/         # HTML 템플릿 (CSS 금지)
│   ├── goals/my-goals.html
│   └── team/tasks.html
└── logic/         # JavaScript 로직
    ├── goals/my-goals.js
    └── team/tasks.js
\`\`\`
- **파일명 = 라우트**: `goals/my-goals.html` → `#/goals/my-goals`
- **분리 원칙**: HTML과 JS 완전 분리

### 2. 역할 기반 메뉴

**일반 직원 (EMPLOYEE)**
- 목표: 나의 목표, 팀 목표 보기, 회사 목표 보기
- 실행: 오늘의 업무, 이번 주, 주간 보고서, 월간 통계
- 성장: 나의 성장 맵, 학습 & 개발, 역량 진단
- 평가: 자기평가, 평가 이력

**팀장 (TEAM_LEADER, DEPT_HEAD)** - 직원 메뉴 +
- 팀 관리: 팀원 업무 현황, 팀 실행 현황, 팀원 주간보고서
- 성장: 팀원 성장 지원
- 평가: 팀원 평가

## BSC (Balanced Scorecard) 관점

회사 목표는 4개 관점으로 분류:
- **재무** (primary) - 매출, 수익성
- **고객** (success) - 만족도, 유지율
- **프로세스** (warning) - 효율성, 품질
- **학습과성장** (info) - 교육, 혁신

팀/개인 목표는 반드시 회사 KPI와 연계 (`companyKPIId`)

## CSS 규칙

### 절대 원칙
❌ **HTML 파일에 `<style>` 태그 절대 금지**
✅ 모든 CSS는 `css/base.css`에 작성

### Bootstrap 우선
\`\`\`html
<!-- ✅ 올바름 -->
<div class="d-flex gap-3 mb-4">
  <div class="col-12 col-md-6">

<!-- ❌ 금지 -->
<div style="display: flex; gap: 12px;">
\`\`\`

## 주의사항

- ✅ `layout: 'default'` 사용 (null 사용 금지)
- ✅ `:key`는 고유 ID 사용 (index 금지)
- ✅ async/await 사용 (Promise then/catch 금지)
- ✅ `@submit.prevent` 폼 제출
- ✅ 모든 경로는 hash 모드 (`#/...`)

## 주요 경로

AI가 참조하는 핵심 경로입니다:

- `src/views/goals/` — 목표 화면
- `src/logic/goals/` — 목표 로직
- `src/views/team/` — 팀 관리 화면
- `src/logic/team/` — 팀 관리 로직
- `src/views/layout/default.html` — 메인 레이아웃

---
**마지막 업데이트:** 2024-01-19
**개발 규칙:** `.claude/rules/` 폴더 참조
```

### 예제 2: Workers 프로젝트

**파일: `CLAUDE.md`**

```markdown
# Cloudflare Workers API 템플릿

## 프로젝트 개요
Cloudflare Workers 기반 REST API 서버 템플릿. JWT 인증, D1 데이터베이스, KV 캐시를 포함한 표준 백엔드 구조.

**기술 스택:** Hono + JWT + Cloudflare Workers (D1, KV, R2)

## 프로젝트 구조

\`\`\`
src/
├── routes/          # API 라우트 핸들러
│   ├── auth.js      # 인증 관련 엔드포인트
│   └── users.js     # 사용자 관련 엔드포인트
├── services/        # 비즈니스 로직 (클래스 기반)
│   ├── authService.js
│   └── userService.js
├── middleware/      # 미들웨어
│   ├── auth.js      # JWT 인증
│   └── errorHandler.js
└── index.js         # 엔트리 포인트
\`\`\`

## 아키텍처 패턴

### 레이어 구조
\`\`\`
Request → Route → Service → Bindings → Response
\`\`\`

### 서비스 레이어 (필수)
모든 서비스는 **클래스 기반**으로 작성:

\`\`\`javascript
export class UserService {
  constructor(env) {
    this.env = env;
  }

  async getUser(userId) {
    // 비즈니스 로직
  }
}
\`\`\`

### 라우트 패턴
\`\`\`javascript
import { UserService } from '../services/userService.js';

users.get('/:id', async (c) => {
  const userId = c.req.param('id');
  const userService = new UserService(c.env);
  const user = await userService.getUser(userId);
  return c.json({ data: user });
});
\`\`\`

## 코딩 컨벤션

### 파일명
- **서비스**: `camelCase` (예: `authService.js`)
- **라우트**: `camelCase` (예: `users.js`)
- **클래스**: `PascalCase` (예: `class AuthService`)
- **상수**: `UPPER_SNAKE_CASE`

### 필수 규칙
1. **서비스는 항상 클래스로 작성**
2. **라우트에는 비즈니스 로직 금지**
3. **env는 생성자에서 주입**
4. **Cloudflare 바인딩 직접 사용** (c.env.KV, c.env.DB, c.env.BUCKET)

## Cloudflare 바인딩 사용

### KV (Key-Value Storage)
\`\`\`javascript
await c.env.KV.put('key', 'value', { expirationTtl: 3600 });
const value = await c.env.KV.get('key', { type: 'json' });
await c.env.KV.delete('key');
\`\`\`

### D1 (SQLite Database)
\`\`\`javascript
const user = await c.env.DB
  .prepare('SELECT * FROM users WHERE id = ?')
  .bind(userId)
  .first();
\`\`\`

### R2 (Object Storage)
\`\`\`javascript
await c.env.BUCKET.put('file.txt', 'Hello World');
const object = await c.env.BUCKET.get('file.txt');
\`\`\`

## 인증 및 보안

### 공개 경로 설정
\`\`\`javascript
// src/index.js
const PUBLIC_PATHS = ['/health', '/docs', '/auth'];
\`\`\`

### 인증된 사용자 정보 접근
\`\`\`javascript
users.get('/profile', async (c) => {
  const userId = c.get('userId');
  const userEmail = c.get('userEmail');
  const userRole = c.get('userRole');
  // ...
});
\`\`\`

## 주의사항

- ✅ 서비스는 클래스로, utils는 함수로
- ✅ 라우트에는 비즈니스 로직 금지
- ✅ 파일명은 camelCase
- ✅ Cloudflare 바인딩은 직접 사용
- ✅ 인증이 필요한 라우트는 PUBLIC_PATHS에서 제외

---
**마지막 업데이트:** 2024-01-24
**상세 가이드:** [CONTRIBUTING.md](CONTRIBUTING.md) 참조
```

### 예제 3: 맑은프레임워크 프로젝트

**파일: `CLAUDE.md`**

```markdown
# 맑은프레임워크 프로젝트

## 프로젝트 개요
맑은프레임워크 기반 웹 애플리케이션. JSP + MyBatis를 사용한 전통적인 Java 웹 애플리케이션으로, 템플릿 엔진을 통한 뷰 분리 패턴을 적용.

**기술 스택:** Java + JSP + MyBatis + 맑은프레임워크 1.14.0

## 프로젝트 구조

\`\`\`
public_html/
├── WEB-INF/
│   ├── config.xml          # 프레임워크 설정
│   └── lib/                # 라이브러리
├── init.jsp                # 전역 초기화
├── main/                   # 메인 JSP 페이지
│   ├── index.jsp
│   └── apply.jsp
├── member/                 # 회원 JSP 페이지
│   ├── login.jsp
│   └── register.jsp
└── html/                   # HTML 템플릿
    ├── main/
    └── member/

src/
└── dao/                    # DAO 클래스
    ├── UserDao.java
    └── BoardDao.java
\`\`\`

## 핵심 원칙

### 1. JSP와 HTML 완전 분리

**❌ 금지:**
\`\`\`jsp
<%
while(list.next()) {
%>
    <div>HTML 마크업</div>
<% } %>
\`\`\`

**✅ 올바름:**

JSP:
\`\`\`jsp
<%
UserDao user = new UserDao();
DataSet list = user.find();
p.setBody("main.user_list");
p.setLoop("users", list);
p.display();
%>
\`\`\`

HTML (`/html/main/user_list.html`):
\`\`\`html
<!--@loop(users)-->
<div>{{users.name}}</div>
<!--/loop(users)-->
\`\`\`

### 2. try-catch 사용 금지

프레임워크가 예외를 처리합니다.

\`\`\`jsp
// ✅ 올바름
if(user.insert()) {
    m.jsAlert("성공");
} else {
    m.jsAlert(user.getErrMsg());
}

// ❌ 금지
try {
    user.insert();
} catch(Exception e) {
    // ...
}
\`\`\`

### 3. Postback 패턴

등록/수정은 같은 JSP에서 처리:

\`\`\`jsp
<%
if(m.isPost() && f.validate()) {
    // POST 처리
    if(user.insert()) {
        m.jsReplace("list.jsp");
    }
    return;
}

// GET 처리 (폼 표시)
p.setBody("main.user_form");
p.display();
%>
\`\`\`

## 파라미터 처리

- **GET**: `m.rs()`, `m.ri()` - XSS 필터 자동
- **POST**: `f.get()` - 원본 데이터

\`\`\`jsp
String keyword = m.rs("keyword");  // GET (검색어)
String content = f.get("content"); // POST (에디터 내용)
\`\`\`

## Page 메소드 호출 순서

\`\`\`jsp
p.setLayout("default");         // 1
p.setBody("main.content");      // 2
p.setVar("title", "제목");       // 3
p.setLoop("list", dataSet);     // 4
p.display();                    // 5
\`\`\`

## 주의사항

- ✅ JSP에 HTML 금지
- ✅ try-catch 금지
- ✅ POST 후 반드시 return
- ✅ DataSet 사용 전 next() 호출
- ✅ 날짜는 VARCHAR(14) + m.time()

---
**개발 규칙:** [docs/coding-principles.md](docs/coding-principles.md) 참조
```

## CLAUDE.md 관리 팁

### 1. 주기적으로 업데이트

프로젝트가 발전하면 CLAUDE.md도 함께 업데이트하세요.

### 2. 너무 길지 않게

500줄 이하로 유지하세요. 너무 길면 Claude Code가 중요한 정보를 놓칠 수 있습니다.

### 3. 핵심 정보만 포함

자세한 내용은 `.claude/rules/`나 별도 문서에 작성하고, CLAUDE.md는 개요와 핵심만 담으세요.

### 4. 실제 코드 예제 사용

이론보다 실제 작동하는 코드 예제가 더 효과적입니다.

### 5. 작업 이력 관리 (선택)

장기 프로젝트는 하단에 작업 이력을 기록하면 Claude Code가 컨텍스트를 더 잘 이해합니다.

```markdown
## 작업 이력

### 2024-01-24
- 팀원 주간보고서 페이지 완성
- 피드백 작성 기능 추가
- 제출 현황 통계 추가

### 2024-01-19
- 팀 목표 관리 페이지 완성
- BSC 관점별 목표 분류 적용
```

## Claude Code가 CLAUDE.md를 활용하는 방법

1. **대화 시작 시**: CLAUDE.md를 자동으로 읽어 프로젝트 전체 파악
2. **코드 작성 시**: 아키텍처 패턴과 규칙을 참고하여 일관된 코드 생성
3. **질문 답변 시**: 프로젝트 구조를 이해하고 정확한 경로와 파일명 제시
4. **디버깅 시**: 데이터 구조와 흐름을 이해하고 문제 파악

## 다음 단계

- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙 정의
- [.claude/templates 활용법](claude-templates.md) - 자주 사용하는 프롬프트 템플릿
- [MCP 서버 설정 및 활용](mcp-setup.md) - 사내 지식 베이스 연동

---

[← 목차로 돌아가기](../_sidebar.md)
