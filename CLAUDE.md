# 바이브코딩 가이드 (vibecoding2)

## 프로젝트 개요

맑은소프트 공식 바이브코딩 표준 매뉴얼. Claude Code를 활용한 실전 개발 가이드를 Docsify 기반 문서 사이트로 제공한다.

**대상 플랫폼 3가지:**
- **맑은프레임워크** - JSP/Java 웹 개발 (맑은소프트 자체 프레임워크)
- **Cloudflare Pages** - ViewLogic Router 기반 SPA
- **Cloudflare Workers** - Hono 기반 서버리스 API

## 기술 스택

- **Docsify 4** - 마크다운 기반 문서 사이트 (빌드 불필요)
- **index.html** - Docsify 설정 및 플러그인 (검색, 코드복사, 페이지네이션)
- **_sidebar.md** - 전체 사이드바 네비게이션
- **docs/** - 모든 문서 파일 (.md)
- **한국어** 전용 문서

## 프로젝트 구조

```
vibecoding2/
├── index.html          # Docsify 진입점 (설정, 플러그인, 테마)
├── _sidebar.md         # 사이드바 네비게이션 (문서 목차)
├── README.md           # 메인 페이지 (Docsify 홈)
├── VIEWLOGIC.md        # ViewLogic Router 사용 매뉴얼
├── docs/               # 모든 가이드 문서
│   ├── quick-start.md
│   ├── setup.md
│   ├── first-project.md
│   ├── claude-md.md         # CLAUDE.md 작성 가이드
│   ├── claude-rules.md      # .claude/rules 작성 가이드
│   ├── claude-memory.md     # .claude/memory 활용법
│   ├── claude-commands.md   # .claude/commands 활용법
│   ├── claude-templates.md  # .claude/templates 활용법
│   ├── mcp-setup.md         # MCP 서버 설정
│   ├── project-structure.md # 프로젝트 구조 표준
│   ├── coding-rules.md      # 코딩 규칙
│   ├── security.md          # 보안 가이드라인
│   ├── best-practices.md    # 베스트 프랙티스
│   ├── code-review-checklist.md
│   ├── context-management.md
│   ├── malgn-*.md           # 맑은프레임워크 가이드
│   ├── pages-*.md           # Cloudflare Pages 가이드
│   └── workers-*.md         # Cloudflare Workers 가이드
└── .claude/
    └── memory/              # 작업 이력
        ├── status.md        # 현재 진행 상태
        ├── history.md       # 작업 이력
        └── next-tasks.md    # 다음 할 일
```

## 문서 카테고리

| 카테고리 | 파일 패턴 | 설명 |
|---------|----------|------|
| 기초 | quick-start, setup, first-project | Claude Code 입문 |
| 설정 | claude-*, mcp-setup, project-structure | .claude 폴더 구성법 |
| 규칙 | coding-rules, security, best-practices | 코딩 표준 |
| 맑은프레임워크 | malgn-* | JSP/Java 개발 가이드 |
| Pages | pages-* | ViewLogic SPA 가이드 |
| Workers | workers-* | Hono API 가이드 |

## 핵심 규칙

### 문서 작성 원칙
- 모든 문서는 **한국어**로 작성
- ✅/❌ 패턴으로 올바른/금지 예제를 대비하여 제시
- 코드 블록에 언어 태그 필수 (```javascript, ```html, ```jsp 등)
- 각 문서 하단에 "다음 단계" 링크와 "← 목차로 돌아가기" 포함
- 문서 간 상대 경로 링크 사용 (Docsify 기준)

### Docsify 구조 규칙
- 새 문서 추가 시 반드시 `_sidebar.md`에도 등록
- README.md = Docsify 메인 페이지 (홈)
- 문서 파일은 모두 `docs/` 폴더에 배치
- 플랫폼별 문서는 접두사 구분: `malgn-`, `pages-`, `workers-`

### 플랫폼별 핵심 개념

**맑은프레임워크:**
- JSP와 HTML 완전 분리 (JSP: 로직, HTML: 템플릿)
- try-catch 금지, boolean 리턴으로 에러 처리
- POST 처리 후 return 필수
- Page 메소드 순서: setLayout → setBody → setVar → setLoop → display

**Cloudflare Pages (ViewLogic):**
- views/ (HTML) + logic/ (JS) 파일 쌍으로 페이지 구성
- 파일명 = 라우트 (`goals/my-goals.html` → `#/goals/my-goals`)
- HTML 파일에 `<style>`, `<script>` 태그 절대 금지
- Bootstrap 5 클래스 우선, CSS는 `css/base.css`에만 작성

**Cloudflare Workers (Hono):**
- 레이어 구조: Request → Route → Service → D1/KV → Response
- 서비스는 반드시 클래스 기반 (`constructor(env)`)
- 라우트에 비즈니스 로직 금지
- Cloudflare 바인딩 직접 사용 (c.env.DB, c.env.KV)

## 참조 프로젝트 (템플릿)

- 맑은프레임워크: `g:\workspace\malgn-template`
- Pages (ViewLogic): `g:\workspace\performance` / viewlogic-template
- Workers: `g:\workspace\workers-template`

## 현재 진행 상태

**완료 (50%):** 기초, 설정, 규칙, 프롬프트 활용법 섹션
**진행 중:** 맑은프레임워크, Pages, Workers 플랫폼별 실전 가이드
**대기 중:** Claude Code 도구 활용, 문제 해결 섹션

상세 현황: `.claude/memory/status.md` 참조

---
**개발 규칙:** `.claude/rules/` 폴더 참조
