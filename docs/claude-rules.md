# .claude/rules 작성 가이드

## 왜 rules가 필요한가?

CLAUDE.md는 세션 시작 시 컨텍스트에 로드되지만, **대화가 길어지면 컨텍스트 압축으로 인해 내용이 밀려날 수 있습니다.** AI가 초반에는 잘 지키던 규칙을 세션 후반에 잊어버리는 경험을 해보셨다면, 이것이 원인입니다.

`.claude/rules/`는 다르게 동작합니다:

| | CLAUDE.md | .claude/rules/ |
|--|-----------|----------------|
| 로드 시점 | 세션 시작 시 1회 | **관련 파일을 다룰 때마다 반복 주입** |
| 세션이 길어지면 | 컨텍스트에서 밀려날 수 있음 | **매번 다시 주입되므로 잊지 않음** |
| 적합한 내용 | 프로젝트 개요, 기술 스택, 폴더 구조 | **절대 어겨서는 안 되는 코딩 규칙** |

**결론: "이건 반드시 지켜야 한다"는 규칙은 rules에, "이런 프로젝트다"는 맥락은 CLAUDE.md에 넣으세요.**

## 적정 규모

rules는 많을수록, 길수록 좋은 게 아닙니다. **매번 컨텍스트에 주입되기 때문에 과하면 오히려 독이 됩니다.**

### 권장 기준

| 항목 | 권장 | 이유 |
|------|------|------|
| 파일 수 | **2~4개** | 너무 많으면 AI가 우선순위를 판단하기 어려움 |
| 파일당 줄 수 | **20~30줄** | 50줄 넘으면 핵심이 묻힘 |
| 전체 규칙 수 | **10~20개** | 규칙이 많을수록 충돌 가능성 증가 |
| 한 규칙의 길이 | **1줄** | 설명이 필요하면 규칙이 아니라 가이드 |

### 너무 많으면 생기는 문제

- 규칙끼리 충돌할 수 있습니다 (A 규칙은 "간결하게", B 규칙은 "상세하게")
- AI가 모든 규칙을 동시에 만족시키려다 오히려 어색한 코드를 생성합니다
- 매 파일 편집마다 큰 규칙이 주입되면 실제 작업에 쓸 컨텍스트가 줄어듭니다

## 어떤 내용을 넣어야 하는가?

### 넣어야 하는 것: AI가 반복적으로 실수하는 패턴

rules에 넣을 가치가 있는 규칙은 **"AI가 알아서 하면 틀릴 가능성이 높은 것"**입니다.

✅ **좋은 규칙 (rules에 넣기):**
- JSP에 HTML 코드 직접 작성 금지 (AI가 자주 섞어서 생성)
- Options API만 사용, `<script setup>` 금지 (AI가 최신 문법을 선호)
- `process.env` 사용 금지, `c.env` 사용 (Cloudflare Workers 고유 패턴)
- POST 처리 후 반드시 `return` (빠뜨리기 쉬운 패턴)

### 넣지 말아야 하는 것: AI가 이미 잘 하는 것

❌ **나쁜 규칙 (넣을 필요 없음):**
- "변수명은 의미있게 작성" → AI가 기본적으로 잘 함
- "들여쓰기는 2칸" → 에디터 설정으로 해결
- "주석을 적절히 작성" → 모호해서 규칙으로 기능하지 않음
- "에러 처리를 꼼꼼히" → 구체적이지 않은 규칙은 무의미

### 판단 기준

규칙을 추가하기 전에 자문하세요:

1. **AI가 이 규칙 없이도 올바르게 코딩할 수 있는가?** → Yes면 불필요
2. **AI가 이 규칙을 어겼을 때 즉시 문제가 되는가?** → No면 가이드 수준 (CLAUDE.md에)
3. **이 규칙은 1줄로 표현할 수 있는가?** → No면 규칙이 아니라 설명

## 잘 쓴 규칙 vs 못 쓴 규칙

### ❌ 못 쓴 규칙

```markdown
# 코딩 규칙

- 코드는 깔끔하게 작성하세요
- 적절한 에러 처리를 해주세요
- 성능을 고려하세요
- 보안에 신경 쓰세요
- 테스트를 작성하세요
```

→ 모호하고, AI가 이미 기본적으로 하는 것들. 규칙으로서 효과 없음.

### ✅ 잘 쓴 규칙

```markdown
# Vue Zero 프론트엔드 규칙

## 필수
- Options API만 사용 (data, methods, mounted, computed)
- Bootstrap 5 클래스로 스타일링, 커스텀 CSS 최소화
- 페이지 추가 후 반드시 pages.json에 등록

## 금지
- <script setup>, Composition API, TypeScript 사용 금지
- <style> 태그 사용 금지
- 404.vue를 pages.json에 등록 금지
```

→ 구체적이고, AI가 실수할 수 있는 포인트만 짚음.

## 파일 구성 예시

프로젝트당 2~4개 파일로 충분합니다:

### Vue Zero 프로젝트

```
.claude/rules/
├── frontend.md     # 프론트엔드 (vue-zero) 규칙
└── backend.md      # 백엔드 (Hono) 규칙
```

**frontend.md:**

```markdown
# 프론트엔드 규칙

## 필수
- Options API만 사용 (data, methods, mounted, computed)
- Bootstrap 5 클래스로 스타일링
- 페이지 추가 시 pages.json 등록 (또는 npm run scan)
- 컴포넌트 추가 시 components.json 등록

## 금지
- <script setup>, Composition API, TypeScript 금지
- <style>, <style scoped> 태그 금지
- 404.vue를 pages.json에 등록 금지
```

**backend.md:**

```markdown
# 백엔드 규칙

## 필수
- 레이어 구조: Route(api/) → DAO(dao/) → D1/KV
- DAO는 클래스 기반, constructor(env)로 바인딩 주입
- 환경변수는 c.env 사용
- 에러는 named error로 throw (ValidationError, NotFoundError)

## 금지
- api/ 파일에 비즈니스 로직 작성 금지
- process.env 사용 금지
- api/ 파일에서 직접 DB 쿼리 금지
```

### 맑은프레임워크 프로젝트

```
.claude/rules/
├── jsp-rules.md    # JSP 규칙
└── html-rules.md   # HTML 템플릿 규칙
```

## 관련 문서

- [CLAUDE.md 작성 가이드](claude-md.md) — 프로젝트 맥락 정보
- [.claude/hooks 활용](claude-hooks.md) — 자동 검증
- [.claude/templates 활용](claude-templates.md) — 코드 생성 템플릿

---

[← 목차로 돌아가기](../_sidebar.md)
