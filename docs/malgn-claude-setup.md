# 맑은프레임워크 .claude 설정

## 개요

맑은프레임워크 프로젝트는 MCP 서버를 활용하여 코딩 규칙과 패턴을 관리합니다. `.claude/rules/`에 모든 규칙을 직접 작성하는 대신, MCP 도구를 통해 동적으로 참조합니다.

## 설정 파일 구조

```
malgn-project/
├── .claude/
│   ├── settings.json          # 권한, 훅 설정
│   ├── rules/
│   │   └── malgn.md           # MCP 참조 규칙
│   └── hooks/
│       └── post-write.sh      # 자동 검증 훅
├── .mcp.json                  # MCP 서버 설정
└── CLAUDE.md                  # 프로젝트 컨텍스트
```

## 1. `.mcp.json`

MCP 서버 연결 설정. 프로젝트 클론 시 자동 적용됩니다.

```json
{
  "mcpServers": {
    "malgn": {
      "type": "http",
      "url": "https://malgn-mcp.apiserver.kr/mcp"
    }
  }
}
```

## 2. `.claude/settings.json`

권한과 훅을 설정합니다.

```json
{
  "permissions": {
    "allow": [
      "Bash(ant compile:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "mcp__malgn__*"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)",
      "Bash(git checkout .:*)",
      "Bash(git clean -f:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/post-write.sh\""
          }
        ]
      }
    ]
  }
}
```

**권한 설명:**
- `ant compile`: DAO 컴파일 자동 허용
- `git` 명령어: 기본 git 작업 허용
- `mcp__malgn__*`: MCP 도구 전체 자동 허용
- 파괴적 git 명령어는 거부

**훅 설명:**
- 파일 작성/편집 시 자동으로 `post-write.sh` 실행
- 코딩 규칙 위반을 자동 체크

## 3. `.claude/rules/malgn.md`

규칙 상세는 MCP에 위임하고, 최소한의 절대 규칙만 명시합니다.

```markdown
# 맑은프레임워크 핵심 규칙

이 프로젝트는 맑은프레임워크(JSP) 기반이다. 상세 규칙/패턴/클래스 정보는
MCP 도구(get_context, get_pattern, get_class, validate_code 등)로 조회할 것.

## 코딩 시 MCP 활용
- 작업 시작: get_context(task, table_name) 로 규칙+패턴+클래스 일괄 조회
- 코드 완성 후: validate_code(code, file_type) 로 규칙 위반 검증

## 절대 규칙
- JSP 시작: <%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%
- JSP에 <%@ page import %> 금지, HTML 직접 작성 금지, <%= %> 금지, try-catch 금지
- JSP는 로직만, HTML은 /html/ 폴더 템플릿으로 완전 분리
- Page 순서: setLayout → setBody → setVar → setLoop → display
- DAO 변수명: tb_ 제거 후 소문자 (UserDao user), DataSet: 단일=info, 복수=list
- GET: m.rs()/m.ri(), POST: f.get()/f.getInt() — request.getParameter() 금지
- null 체크 불필요 (프레임워크가 빈 문자열 반환)
- SQL 바인딩 필수: find("id = ?", new Object[]{id})
- POST 처리 후 return 필수
- m, f, p, j, auth, isLogin, userId 는 init.jsp에서 자동 초기화됨
```

## 4. `CLAUDE.md`

프로젝트 개요와 구조를 간결하게 기술합니다.

```markdown
# 맑은프레임워크 템플릿 프로젝트

JSP 기반 맑은프레임워크(malgn framework) 웹 애플리케이션 템플릿.

## 기술 스택
- 백엔드: JSP/Servlet, 맑은프레임워크 (malgn.jar)
- 프론트: Bootstrap 5, HTML 템플릿 엔진
- DB: MySQL (JNDI 연결)
- API: RESTful + JWT 인증
- 빌드: Ant (ant compile)

## 프로젝트 구조
src/dao/              DAO 클래스 (Java) → ant compile로 빌드
public_html/
  init.jsp            공통 초기화 (m, f, p, j, auth 자동 생성)
  {기능}/              JSP (로직만)
  html/layout/        레이아웃 HTML (layout_xxx.html)
  html/{기능}/         본문 HTML 템플릿
  api/init.jsp        API 초기화 (JWT, CORS)
  api/{기능}.jsp       REST API 엔드포인트
  WEB-INF/config.xml  프레임워크 설정
schema.sql            DB 스키마

## 작업 워크플로우
1. MCP get_context(task, table_name) 로 규칙/패턴/클래스 조회
2. MCP get_pattern(type) 으로 표준 패턴 참조하여 코딩
3. MCP validate_code(code, file_type) 로 규칙 위반 검증
4. DAO 수정 시 ant compile 로 컴파일

## MCP 도구 (malgn)
- get_context — 작업별 규칙+패턴+클래스 일괄 조회
- get_pattern — 코드 패턴 템플릿
- get_class — 클래스 메소드 상세 조회
- get_rules — 코딩 규칙 조회
- validate_code — 코드 규칙 위반 검증
- get_doc / search_docs — 프레임워크 문서 조회
```

## 5. 슬래시 커맨드

`.claude/commands/` 폴더에 자주 사용하는 작업을 커맨드로 정의합니다.

| 커맨드 | 파일 | 설명 |
|--------|------|------|
| `/project:crud` | `crud.md` | CRUD 전체 생성 |
| `/project:api` | `api.md` | REST API 생성 |
| `/project:new-page` | `new-page.md` | 단일 페이지 생성 |
| `/project:schema` | `schema.md` | 테이블 스키마 생성 |
| `/project:validate` | `validate.md` | 코드 규칙 검증 |
| `/project:review` | `review.md` | 코드 리뷰 |

커맨드 내에서 MCP 도구를 호출하도록 작성하면, 규칙을 자동으로 준수하는 코드가 생성됩니다.

## MCP 기반 접근의 장점

- rules 파일을 최소화 (토큰 절약)
- 규칙 업데이트 시 MCP 서버만 수정 (프로젝트 코드 변경 불필요)
- 모든 프로젝트에 동일한 규칙 자동 적용
- 코드 검증까지 자동화

---

[← 목차로 돌아가기](../_sidebar.md)
