# 바이브코딩 가이드

> Claude Code와 함께하는 실전 개발 가이드

## 왜 바이브코딩인가?

AI 코딩 도구가 보편화되면서 **"어떻게 물어봐야 하는가?"**가 개발 생산성의 핵심이 되었습니다.
하지만 많은 개발자들이 여전히 시행착오를 반복하며 AI를 활용하고 있습니다.

**바이브코딩**은 Claude Code를 활용한 **실전 개발 노하우**를 체계화한 가이드입니다.

## 핵심 개념

### 1. 프롬프트 패턴
단순히 "이것 좀 만들어줘"가 아닌, **구조화된 요청**으로 정확한 결과를 얻습니다.

```
❌ "게시판 만들어줘"
✅ "게시판 CRUD API를 만들어줘.
    - 경로: /api/posts
    - D1 데이터베이스 사용
    - 페이징 10개씩
    - 검색 기능 (제목+내용)"
```

### 2. 컨텍스트 관리
AI가 프로젝트를 정확히 이해하도록 **`.claude/` 폴더**로 규칙을 정의합니다.

```
.claude/
├── rules/          # 코딩 규칙
├── templates/      # 코드 템플릿
└── CLAUDE.md       # 프로젝트 개요
```

### 3. 플랫폼별 최적화
맑은프레임워크, Cloudflare Pages/Workers 등 **각 플랫폼의 특성**에 맞춘 개발 방식을 제시합니다.

## 이 가이드의 가치

### 시행착오 제거
수십 번의 재시도 대신 **첫 시도부터 정확한 결과**를 얻습니다.

### 일관성 유지
팀 전체가 **동일한 코딩 스타일과 패턴**을 유지합니다.

### 학습 곡선 단축
신규 개발자도 **즉시 프로덕션 코드**를 작성할 수 있습니다.

## 빠른 시작

### 1. 5분 안에 시작하기
```
Claude Code를 열고 물어보세요:

"맑은프레임워크 프로젝트를 새로 만들어줘.
 docs/quick-start.md를 참고해서 기본 구조 생성."
```

### 2. 첫 프로젝트 만들기
- [맑은프레임워크](docs/malgn-getting-started.md) - JSP 기반 웹 개발
- [Cloudflare Pages](docs/pages-getting-started.md) - 정적 사이트/SPA
- [Cloudflare Workers](docs/workers-getting-started.md) - 서버리스 API

## 문서 구조

### 🚀 기초
- [빠른 시작](docs/quick-start.md) - 5분 안에 Claude Code 활용
- [개발 환경 설정](docs/setup.md)
- [첫 프로젝트 만들기](docs/first-project.md)

### ⚙️ 설정
- [.claude/rules 작성](docs/claude-rules.md) - 코딩 규칙 정의
- [CLAUDE.md 작성](docs/claude-md.md) - 프로젝트 개요
- [MCP 서버 설정](docs/mcp-setup.md) - 사내 문서 연동

### 📝 프롬프트 패턴
- [자주 사용하는 패턴](docs/prompt-patterns.md)
- [실전 시나리오 18가지](docs/common-scenarios.md)
- [효과적인 프롬프트 작성 팁](docs/effective-prompts.md)

### 🛠️ 플랫폼별 가이드
- **맑은프레임워크** (8개 문서) - JSP/Java 웹 개발
- **Cloudflare Pages** (8개 문서) - 정적 사이트/SPA
- **Cloudflare Workers** (8개 문서) - 서버리스 API

### 🔧 도구 활용
- [파일 작업](docs/file-operations.md) - Read, Write, Edit
- [코드 탐색](docs/code-navigation.md) - Grep, Glob, Task
- [Git 자동화](docs/git-automation.md) - 커밋, PR 생성

### 🐛 문제 해결
- [일반적인 오류](docs/troubleshooting.md)
- [성능 최적화](docs/performance.md)
- [디버깅 전략](docs/debugging.md)

## 실전 예제

### API 개발 (Workers)
```
사용자 CRUD API를 만들어줘.

엔드포인트:
- GET /api/users - 목록 (페이징 10개)
- GET /api/users/:id - 상세
- POST /api/users - 생성
- PUT /api/users/:id - 수정
- DELETE /api/users/:id - 삭제

D1 데이터베이스 사용.
JWT 인증 필요 (POST/PUT/DELETE).
```

### 페이지 개발 (맑은프레임워크)
```
게시판 목록 페이지를 만들어줘.

경로: /board/board_list.jsp
기능:
- 페이징 (10개씩)
- 검색 (제목+내용)
- 정렬 (최신순)

ListManager 사용.
작성자 이름도 함께 표시 (JOIN).
```

## 주요 특징

✅ **실전 중심** - 바로 사용 가능한 예제와 패턴
✅ **플랫폼 특화** - 맑은프레임워크, Pages, Workers 상세 가이드
✅ **체크리스트** - 개발 시 빠뜨리지 말아야 할 사항
✅ **문제 해결** - 자주 발생하는 오류와 해결 방법
✅ **한국어** - 한국 개발 환경에 최적화

## 기여하기

이 가이드는 실전 경험을 기반으로 지속적으로 업데이트됩니다.

- 새로운 패턴 제안
- 오류 수정
- 예제 추가

Pull Request를 환영합니다!

## 라이선스

이 문서는 실전 개발을 위한 가이드로 자유롭게 사용하실 수 있습니다.

---

**바이브코딩으로 개발 생산성을 10배 높이세요!** 🚀
