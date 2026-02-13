# MCP 서버 설정 및 활용

## MCP란?

**MCP (Model Context Protocol)**는 AI 모델이 외부 데이터 소스에 접근할 수 있게 해주는 프로토콜입니다. Claude Code에서 MCP를 사용하면 사내 문서, 프레임워크 가이드, 코딩 규칙 등을 실시간으로 참조할 수 있습니다.

## 맑은프레임워크 MCP 서버

맑은소프트는 맑은프레임워크 전용 MCP 서버를 운영합니다.

**서버 URL:** `https://malgn-mcp.apiserver.kr/mcp`

### 제공 도구

| 도구 | 설명 | 사용 시점 |
|------|------|-----------|
| `get_context` | 작업별 규칙+패턴+클래스 일괄 조회 | 작업 시작 시 |
| `get_pattern` | 코드 패턴 템플릿 (jsp-list, dao-basic 등) | 코드 작성 시 |
| `get_class` | 클래스 메소드 상세 조회 | API 확인 시 |
| `get_rules` | 코딩 규칙 조회 | 규칙 확인 시 |
| `validate_code` | 코드 규칙 위반 검증 | 코드 완성 후 |
| `get_doc` | 프레임워크 문서 조회 | 문서 참조 시 |
| `search_docs` | 프레임워크 문서 검색 | 키워드 검색 시 |

## 설정 방법

### 방법 1: 프로젝트 레벨 설정 (권장)

프로젝트 루트에 `.mcp.json` 파일을 생성합니다. 이 방법은 프로젝트를 클론하면 MCP가 자동으로 설정되어 팀 전체에 동일한 환경을 제공합니다.

**`.mcp.json`:**

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

### 방법 2: 글로벌 설정

모든 프로젝트에서 MCP를 사용하려면 글로벌 설정 파일에 추가합니다.

**macOS/Linux:**
```
~/.claude/settings.json
```

**Windows:**
```
%USERPROFILE%\.claude\settings.json
```

### 방법 3: Claude Code UI에서 설정

1. Settings 열기 (`Ctrl+,`)
2. MCP Servers 섹션 찾기
3. Edit in JSON 클릭
4. 서버 설정 입력
5. 저장 및 재시작

## 권한 설정

`.claude/settings.json`에서 MCP 도구 사용을 자동 허용할 수 있습니다:

```json
{
  "permissions": {
    "allow": [
      "mcp__malgn__*"
    ]
  }
}
```

이 설정으로 MCP 도구 호출 시 매번 승인하지 않아도 됩니다.

## 워크플로우

MCP를 활용한 표준 작업 흐름:

```
1. 작업 시작 → get_context(task, table_name) 로 규칙/패턴/클래스 조회
2. 코드 작성 → get_pattern(type) 으로 표준 패턴 참조
3. API 확인 → get_class(class_name) 으로 메소드 조회
4. 코드 검증 → validate_code(code, file_type) 으로 규칙 위반 체크
```

### rules에서 MCP 자동 참조

`.claude/rules/malgn.md`에서 MCP 도구를 참조하도록 설정하면, Claude가 코드 작성 시 자동으로 MCP를 호출합니다:

```markdown
# 맑은프레임워크 핵심 규칙

이 프로젝트는 맑은프레임워크(JSP) 기반이다. 상세 규칙/패턴/클래스 정보는
MCP 도구(get_context, get_pattern, get_class, validate_code 등)로 조회할 것.

## 코딩 시 MCP 활용
- 작업 시작: get_context(task, table_name) 로 규칙+패턴+클래스 일괄 조회
- 코드 완성 후: validate_code(code, file_type) 로 규칙 위반 검증
```

### 슬래시 커맨드에서 MCP 활용

`.claude/commands/`에서 MCP 도구를 호출하는 커맨드를 만들 수 있습니다:

```markdown
# /project:crud $TABLE_NAME

1. MCP get_context("crud", "$TABLE_NAME") 로 규칙/패턴 조회
2. schema.sql에서 $TABLE_NAME 테이블 구조 확인
3. get_pattern("jsp-list"), get_pattern("jsp-insert") 등으로 패턴 참조
4. DAO + JSP 5개 + HTML 4개 생성
5. validate_code()로 규칙 위반 검증
6. ant compile로 DAO 컴파일
```

## 연결 확인

Claude Code에서 다음을 입력하여 MCP 연결을 테스트합니다:

```
mcp 연결 테스트
```

7개 도구(get_context, validate_code, get_class, get_rules, get_pattern, get_doc, search_docs)가 모두 정상이면 준비 완료입니다.

## 문제 해결

### MCP 서버가 연결되지 않음

1. `.mcp.json` 파일이 프로젝트 루트에 있는지 확인
2. JSON 문법 오류 확인 (쉼표, 중괄호)
3. Claude Code 재시작
4. 맑은소프트 개발팀에 문의

### 도구가 작동하지 않음

1. `.claude/settings.json`에서 `mcp__malgn__*` 권한 확인
2. MCP 서버 상태 확인 (서버 점검 중일 수 있음)

## 보안 고려사항

- MCP 서버 URL을 외부에 공유 금지
- `.mcp.json`은 Git에 커밋 가능 (공개 URL만 포함)
- `.claude/settings.json`은 프로젝트별 관리

---

[← 목차로 돌아가기](../_sidebar.md)
