# .claude/templates 활용법

## 템플릿이란?

`.claude/templates/` 폴더에 코드 패턴을 마크다운으로 저장하면, AI가 새 기능을 만들 때 이 패턴을 참고하여 일관된 코드를 생성합니다.

## 언제 템플릿이 필요한가?

AI는 기존 코드를 읽고 같은 패턴으로 생성하는 능력이 뛰어납니다. 따라서 **기존 코드가 충분하면 템플릿은 불필요**합니다. 템플릿이 진짜 가치 있는 경우는 따로 있습니다.

### 템플릿이 필요한 경우

| 상황 | 이유 |
|------|------|
| **프로젝트 초기** | 참고할 기존 코드가 아직 없어서 AI가 패턴을 추론할 수 없음 |
| **AI가 추론하기 어려운 복잡한 구조** | 맑은프레임워크의 JSP/HTML 분리처럼 일반적이지 않은 패턴 |
| **회사 고유 코드 규칙** | 일반적인 프레임워크 사용법과 다른 사내 컨벤션 |

### 템플릿이 불필요한 경우

| 상황 | 대안 |
|------|------|
| 이미 같은 패턴의 코드가 프로젝트에 있음 | AI가 기존 코드를 읽고 따라함 |
| 표준적인 CRUD, API, 폼 | 자연어 요청으로 충분 |
| 코딩 규칙 강제 | rules + hooks가 더 적합 |

**판단 기준:** "이미 프로젝트에 비슷한 코드가 2개 이상 있는가?" → Yes면 템플릿 불필요. AI가 기존 코드를 참고합니다.

## 작성 원칙

### 실제 동작하는 코드로 작성

템플릿은 설명이 아니라 **복사해서 바로 쓸 수 있는 코드**여야 합니다.

✅ **좋은 템플릿:**
```markdown
# 맑은프레임워크 목록 페이지

## JSP 파일: /WEB-INF/jsp/{{entity}}/list.jsp
\```jsp
<%@ page contentType="text/html; charset=UTF-8" %>
<%
Page p = new Page(request, response);
p.setLayout("layout/default");
p.setBody("{{entity}}/list");

Dao dao = new Dao("{{table}}");
dao.setS("SELECT * FROM {{table}} ORDER BY reg_date DESC");
if (dao.selectList()) {
    p.setLoop("list", dao);
}
p.display();
%>
\```

## HTML 파일: /html/{{entity}}/list.html
\```html
<div class="container">
  <h2>{{Entity}} 목록</h2>
  <table class="table">
    <!-- {loop:list} -->
    <tr><td>{=name}</td><td>{=reg_date}</td></tr>
    <!-- {/loop:list} -->
  </table>
</div>
\```
```

❌ **나쁜 템플릿:**
```markdown
# 목록 페이지
- JSP에 로직을 작성합니다
- HTML에 템플릿을 작성합니다
- Page 객체로 연결합니다
```

→ 설명만 있고 코드가 없으면 AI가 참고할 수 없습니다.

### 핵심만 포함

템플릿에 모든 경우의 수를 넣으면 오히려 혼란을 줍니다. **가장 기본적인 패턴 하나**를 깔끔하게 보여주세요.

✅ 기본 목록 페이지 1개 → AI가 이를 기반으로 변형
❌ 검색 있는 목록, 페이징 있는 목록, 필터 있는 목록 전부 → 너무 많아서 핵심이 묻힘

### 보안 요소는 기본 포함

템플릿에 보안 패턴을 넣어두면 AI가 자연스럽게 따라합니다:

- 입력 검증
- XSS 방지 (맑은프레임워크: `m.rs()`, `m.ri()`)
- SQL Injection 방지 (바인드 변수)
- 권한 체크

## 플레이스홀더

변경이 필요한 부분은 `{{이름}}` 형식으로 표시합니다:

| 플레이스홀더 | 의미 | 예시 |
|-------------|------|------|
| `{{entity}}` | 리소스명 (소문자) | user, product |
| `{{Entity}}` | 리소스명 (대문자 시작) | User, Product |
| `{{table}}` | 테이블명 | tb_user, products |

간단하게 3개면 충분합니다. AI가 맥락에 맞게 알아서 치환합니다.

## 템플릿 구성 예시

프로젝트당 2~4개면 충분합니다. 프로젝트에 기존 코드가 쌓이면 점차 불필요해집니다.

### 맑은프레임워크

```
.claude/templates/
├── malgn-list.md       # 목록 페이지 (JSP + HTML)
└── malgn-form.md       # 등록/수정 페이지 (JSP + HTML)
```

→ AI가 맑은프레임워크의 JSP/HTML 분리 패턴을 처음 접할 때 필수

### Vue Zero

```
.claude/templates/
└── vue-zero-page.md    # 페이지 + API + DAO 세트
```

→ 프로젝트 초기에만 필요. 기존 페이지가 2~3개 쌓이면 제거해도 됨

## 사용 방법

AI에게 템플릿을 참조하라고 명시합니다:

```
".claude/templates/malgn-list.md를 참고해서 상품 목록 페이지를 만들어줘."
```

프로젝트에 기존 코드가 충분해지면:

```
"기존 사용자 목록 페이지와 같은 패턴으로 상품 목록 페이지를 만들어줘."
```

→ 이 시점이 오면 템플릿은 역할을 다한 것입니다.

## 관련 문서

- [CLAUDE.md 작성](claude-md.md) — 프로젝트 맥락 정보
- [.claude/rules 작성](claude-rules.md) — 코딩 규칙 가이드라인
- [.claude/hooks 활용](claude-hooks.md) — 자동 검증

---

[← 목차로 돌아가기](../_sidebar.md)
