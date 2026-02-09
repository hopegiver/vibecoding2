# .claude/rules 작성 가이드

## 개요

`.claude/rules` 폴더는 Claude Code가 코드 작성 시 **반드시 따라야 할 규칙**을 정의합니다.

## 기본 규칙

### 파일 구조
- `.claude/rules/` 폴더 안의 모든 `.md` 파일이 자동 로드
- 파일명은 자유롭게 지정 가능
- 권장: architecture.md, coding-conventions.md, security.md

### 크기 제한
- 개별 파일: 50줄 이내
- 한 문장씩 간결하게 작성
- 장황한 설명 금지

### 작성 원칙
- 필수 사항과 금지 사항으로 구분
- 목록 형태로 작성
- 예제 코드는 최소한만 포함

### 파일 확장자별 규칙 적용

규칙 파일 상단에 "적용 대상" 명시:

```markdown
# JavaScript 규칙

적용 대상: .js, .ts, .jsx, .tsx

## 필수 사항
- const, let 사용 (var 금지)
...
```

Claude Code는 파일 작업 시 해당 확장자의 규칙을 자동으로 참조합니다.

## 실전 예제

### Workers 프로젝트

**파일: `.claude/rules/architecture.md`**

```markdown
# Workers 아키텍처

## 필수 사항
- 레이어 구조: Request → Route → Service → D1/KV
- 서비스는 클래스 기반으로 작성
- KV 캐시 우선 사용, D1은 백업
- 비동기 함수는 async/await 사용
- 파일명: userService.js, 클래스명: UserService

## 금지 사항
- 라우트에서 직접 DB 접근 금지
- Promise then/catch 사용 금지
- 비즈니스 로직을 라우트에 작성 금지
```

### 맑은프레임워크 프로젝트

**파일: `.claude/rules/core-principles.md`**

```markdown
# 맑은프레임워크 핵심 규칙

## 필수 사항
- JSP와 HTML은 완전히 분리
- JSP에는 로직만, HTML에는 템플릿만
- Page 메소드 순서: setLayout → setBody → setVar → setLoop → display
- POST 처리 후 반드시 return 필수
- GET 파라미터: m.rs(), m.ri() 사용 (XSS 자동 필터)
- POST 파라미터: f.get() 사용 (원본 데이터)

## 금지 사항
- JSP에서 try-catch 사용 금지 (if문으로 처리)
- JSP에 HTML 코드 작성 금지
- 템플릿에 로직(연산, 삼항연산자) 금지
- HTML 파일에 style 태그 금지
```

### Pages 프로젝트

**파일: `.claude/rules/style-guide.md`**

```markdown
# CSS 규칙

## 필수 사항
- Bootstrap 5 클래스 최대 활용
- CSS 변수 사용: --primary-color, --success-color
- 반응형: col-12 col-md-6 col-xl-3
- 모바일 우선: @media (max-width: 768px)

## 금지 사항
- HTML 파일에 style 태그 사용 금지
- Custom CSS 최소화
- !important 사용 금지
```

## 템플릿 활용

예제 코드는 `.claude/templates/` 폴더에 별도로 작성합니다. 규칙 파일에는 필수/금지 사항만 간결하게 나열합니다.

## 다음 단계

- [.claude/templates 활용법](claude-templates.md) - 자주 사용하는 프롬프트 템플릿 만들기
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트 문서 작성
- [MCP 서버 설정 및 활용](mcp-setup.md) - 사내 지식 베이스 연동

---

[← 목차로 돌아가기](../_sidebar.md)
