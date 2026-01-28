# .claude/rules 작성 가이드

## 개요

`.claude/rules` 폴더는 Claude Code가 프로젝트에서 코드를 작성할 때 **반드시 따라야 할 규칙**을 정의하는 곳입니다. 이 폴더에 있는 마크다운 파일들은 Claude Code의 시스템 프롬프트에 자동으로 포함되어, AI가 여러분의 프로젝트 표준을 이해하고 일관된 코드를 작성하게 됩니다.

## 왜 필요한가?

Claude Code는 일반적인 코딩 패턴은 잘 알고 있지만, **여러분 회사만의 특별한 규칙**은 모릅니다:

- 회사 코딩 컨벤션
- 프레임워크 특화 패턴
- 금지된 코딩 방식
- 보안 규칙
- 파일 구조 표준

`.claude/rules`에 이런 규칙을 정의하면, Claude Code가 매번 일관되게 여러분의 표준을 따릅니다.

## 기본 구조

```
.claude/
└── rules/
    ├── coding-conventions.md  # 코딩 컨벤션
    ├── architecture.md        # 아키텍처 패턴
    ├── style-guide.md         # 스타일 가이드
    └── security.md            # 보안 규칙
```

파일명은 자유롭게 지정할 수 있습니다. `.claude/rules/` 폴더 안의 모든 `.md` 파일이 자동으로 로드됩니다.

## 적정 크기 가이드

### 권장 크기
- **개별 파일**: 2,000-3,000 토큰 (약 6,000-9,000자)
- **전체 폴더**: 10,000-15,000 토큰 (약 30,000-45,000자)

### 토큰 수 확인
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer) 사용
- 한글 기준: 글자 수 × 3 ≈ 토큰 수

### 초과 시 문제점
- Claude Code의 응답 속도 저하
- 중요한 규칙을 놓칠 수 있음
- 컨텍스트 압축으로 세부 사항 누락

### 크기 관리 팁
- 규칙을 주제별로 여러 파일로 분리
- 장황한 설명보다 명확한 예제 코드 사용
- 자세한 내용은 별도 문서로 링크

## 작성 원칙

### 1. 명확하고 구체적으로 작성

❌ **나쁜 예:**
```markdown
# 코딩 규칙
- 좋은 코드를 작성하세요
- 성능을 고려하세요
```

✅ **좋은 예:**
```markdown
# 코딩 규칙

## 필수 사항
1. **모든 함수는 camelCase로 작성**
2. **클래스는 PascalCase로 작성**
3. **상수는 UPPER_SNAKE_CASE로 작성**

## 금지 사항
- ❌ `var` 키워드 사용 금지 (const, let만 사용)
- ❌ 익명 함수 사용 금지 (모든 함수는 이름 필요)
```

### 2. 예제 코드 포함

Claude Code는 예제 코드를 보면 더 잘 이해합니다.

```markdown
# API 호출 패턴

## ✅ 올바른 방법
\`\`\`javascript
// 서비스 클래스를 통한 API 호출
const userService = new UserService(env);
const user = await userService.getUser(123);
\`\`\`

## ❌ 잘못된 방법
\`\`\`javascript
// 라우트에서 직접 DB 접근 금지
const user = await env.DB
  .prepare('SELECT * FROM users WHERE id = ?')
  .bind(123)
  .first();
\`\`\`
```

### 3. 우선순위 명시

가장 중요한 규칙을 상단에 배치하세요.

```markdown
# 핵심 개발 규칙

## ⚡ 최우선 원칙 (절대 위반 금지)

1. **HTML 파일에 `<style>` 태그 사용 금지**
2. **JSP에서 try-catch 사용 금지**
3. **템플릿에 로직(연산, 삼항연산자) 넣지 말 것**

## 필수 준수 사항

...

## 권장 사항

...
```

### 4. 이모지로 시각적 구분

```markdown
## ⚡ 최우선 원칙
## ✅ 올바른 예
## ❌ 잘못된 예
## 🚫 절대 금지
## ⚠️ 주의사항
## 💡 팁
```

## 실전 예제

> **중요:** 아래 예제들은 **가이드 문서용 상세 예제**입니다. 실제 `.claude/rules/` 파일은 이보다 **훨씬 간결**하게 작성해야 합니다 (개별 파일 2,000-3,000 토큰 이내).

### 예제 1: Pages 프로젝트 스타일 가이드 (간결 버전)

**파일: `.claude/rules/style-guide.md` (약 800 토큰)**

```markdown
# CSS 핵심 규칙

## ⚡ 최우선 원칙

**Bootstrap 5 최대 활용, Custom CSS 최소화**
- Layout: `d-flex`, `row`, `col-*`, `gap-*`
- Spacing: `p-*`, `m-*`, `mb-3`
- Text: `fw-bold`, `text-center`

## 🚫 절대 금지

**HTML에 `<style>` 태그 금지**
\`\`\`html
<!-- ❌ --> <style>.sidebar { width: 250px; }</style>
<!-- ✅ --> css/base.css에 작성
\`\`\`

## 필수 규칙

1. **CSS 변수 사용**
   - `--primary-color: #6366f1`
   - `--success-color: #10b981`
   - `--danger-color: #ef4444`

2. **반응형**
   - 모바일: `@media (max-width: 768px)`
   - Grid: `col-12 col-md-6 col-xl-3`
```

### 예제 2: Workers 아키텍처 (간결 버전)

**파일: `.claude/rules/architecture.md` (약 1,000 토큰)**

```markdown
# Workers 아키텍처

## 레이어 구조
\`\`\`
Request → Route → Service → Bindings → Response
\`\`\`

## 서비스 패턴 (필수)

\`\`\`javascript
// ✅ 클래스 기반 서비스
export class UserService {
  constructor(env) { this.env = env; }

  async getUser(id) {
    // KV 캐시 → D1 → 캐시 저장
    const cached = await this.env.KV.get(\`user:\${id}\`, { type: 'json' });
    if (cached) return cached;

    const user = await this.env.DB.prepare('SELECT * FROM users WHERE id = ?').bind(id).first();
    if (user) await this.env.KV.put(\`user:\${id}\`, JSON.stringify(user), { expirationTtl: 3600 });
    return user;
  }
}
\`\`\`

## 라우트 패턴

\`\`\`javascript
// ✅ 비즈니스 로직은 서비스에 위임
users.get('/:id', async (c) => {
  const service = new UserService(c.env);
  const user = await service.getUser(c.req.param('id'));
  return c.json({ data: user });
});
\`\`\`

## 🚫 금지

\`\`\`javascript
// ❌ 라우트에서 직접 DB 접근 금지
users.get('/:id', async (c) => {
  const user = await c.env.DB.prepare('...').bind(id).first();
  return c.json({ data: user });
});
\`\`\`

## 파일명
- 서비스: `camelCase` (userService.js)
- 클래스: `PascalCase` (UserService)
```

### 예제 3: 맑은프레임워크 핵심 규칙 (간결 버전)

**파일: `.claude/rules/core-principles.md` (약 1,200 토큰)**

```markdown
# 맑은프레임워크 핵심 규칙

## ⚡ 절대 원칙

### 1. JSP/HTML 완전 분리

\`\`\`jsp
// ❌ JSP에 HTML 금지
<% while(list.next()) { %> <div>...</div> <% } %>

// ✅ JSP: 로직만
<%
DataSet list = user.find();
p.setBody("main.list");
p.setLoop("user", list);
p.display();
%>
\`\`\`

\`\`\`html
<!-- ✅ HTML: 템플릿만 -->
<!--@loop(user)-->
<div>{{user.name}}</div>
<!--/loop(user)-->
\`\`\`

### 2. try-catch 금지

\`\`\`jsp
// ❌ try { user.insert(); } catch(e) { }
// ✅ if(user.insert()) { } else { m.p(user.getErrMsg()); }
\`\`\`

### 3. POST 후 return 필수

\`\`\`jsp
if(m.isPost()) {
    user.insert();
    m.jsReplace("list.jsp");
    return;  // 필수!
}
p.display();
\`\`\`

## 파라미터 처리

- **GET**: `m.rs("keyword")`, `m.ri("page")` - XSS 필터 자동
- **POST**: `f.get("content")` - 원본 데이터

## Page 메소드 순서

\`\`\`jsp
p.setLayout("default");      // 1
p.setBody("main.content");   // 2 (필수)
p.setVar("title", "제목");    // 3
p.setLoop("list", dataSet);  // 4
p.display();                 // 5 (필수)
\`\`\`
```

**권장 크기 요약:**
- 예제 1: ~800 토큰 (약 30줄)
- 예제 2: ~1,000 토큰 (약 40줄)
- 예제 3: ~1,200 토큰 (약 45줄)
- **합계: ~3,000 토큰** (개별 파일 기준 적정 크기)

## 규칙 파일 관리 팁

### 1. 규칙을 작고 명확한 파일로 분리

- `coding-conventions.md` - 네이밍, 포맷팅 등
- `architecture.md` - 레이어 구조, 패턴
- `style-guide.md` - CSS, HTML 규칙
- `security.md` - 보안 관련 규칙
- `api-patterns.md` - API 설계 패턴

### 2. 자주 위반되는 규칙은 상단에 배치

Claude Code가 가장 먼저 읽는 내용이 가장 잘 준수됩니다.

### 3. 예제 코드는 실제 프로젝트 코드 복사

실제로 작동하는 코드를 예제로 사용하면 Claude Code가 더 정확하게 이해합니다.

### 4. 정기적으로 업데이트

프로젝트가 발전하면서 새로운 패턴이 생기면 규칙 파일도 업데이트하세요.

## Claude Code가 규칙을 어떻게 활용하나?

1. **대화 시작 시**: `.claude/rules/` 폴더의 모든 파일을 자동으로 읽습니다
2. **코드 작성 시**: 규칙을 참고하여 일관된 코드를 생성합니다
3. **리뷰 시**: 작성한 코드가 규칙을 따르는지 자동으로 체크합니다

## 다음 단계

- [.claude/templates 활용법](claude-templates.md) - 자주 사용하는 프롬프트 템플릿 만들기
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트 문서 작성
- [MCP 서버 설정 및 활용](mcp-setup.md) - 사내 지식 베이스 연동

---

[← 목차로 돌아가기](../_sidebar.md)
