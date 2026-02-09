# MCP 서버 설정 및 활용

## MCP란?

**MCP (Model Context Protocol)**는 AI 모델이 외부 데이터 소스에 접근할 수 있게 해주는 프로토콜입니다. Claude Code에서 MCP를 사용하면:

- 📚 **사내 위키/문서** 자동 참조
- 🔧 **프레임워크 공식 문서** 실시간 조회
- 🗄️ **데이터베이스 스키마** 자동 인식
- 🌐 **API 스펙** 자동 참조

## 왜 필요한가?

### MCP 없이 작업할 때

```
프롬프트: "맑은프레임워크에서 DataSet 사용법을 알려줘"

Claude Code: "죄송하지만 맑은프레임워크에 대한 정보가 없습니다.
일반적인 Java 패턴으로 설명드릴게요..."
```

### MCP 사용 시

```
프롬프트: "@malgn-docs DataSet 사용법을 참고해서 사용자 목록 조회 코드를 만들어줘"

Claude Code: "맑은프레임워크 문서를 확인했습니다.
DataSet.next()로 반복하는 패턴을 사용하겠습니다..."

// 정확한 코드 생성!
```

## MCP 서버 유형

### 1. 파일 시스템 MCP (로컬 문서)

로컬에 저장된 문서를 참조합니다.

**사용 사례:**
- 사내 코딩 가이드 (PDF, Markdown)
- 프레임워크 매뉴얼
- API 스펙 문서

### 2. HTTP MCP (원격 API)

HTTP API를 통해 문서를 가져옵니다.

**사용 사례:**
- Confluence, Notion 등 사내 위키
- GitHub Wiki
- 공식 문서 사이트

### 3. 데이터베이스 MCP

데이터베이스 스키마를 참조합니다.

**사용 사례:**
- 테이블 구조 자동 인식
- 컬럼명 자동 완성
- 관계 파악

## 설정 방법

### 방법 1: Claude Code UI에서 설정 (권장)

**Step 1: Settings 열기**

1. Claude Code 채팅창 우측 상단 **⚙️ 아이콘** 클릭
2. 또는 `Ctrl+,` (Windows/Linux) / `Cmd+,` (macOS)

**Step 2: MCP Servers 섹션 찾기**

Settings 화면에서 **"MCP Servers"** 또는 **"Model Context Protocol"** 섹션 찾기

**Step 3: 서버 추가**

1. **"Add Server"** 또는 **"Edit in JSON"** 클릭
2. JSON 에디터가 열림

**Step 4: 서버 설정 입력**

**API 서버 방식 (권장):**

```json
{
  "mcpServers": {
    "malgn-docs": {
      "url": "https://mcp.malgnsoft.com/docs",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN"
      }
    },
    "company-wiki": {
      "url": "https://mcp.malgnsoft.com/wiki",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN"
      }
    }
  }
}
```

**파일시스템 방식 (로컬 문서):**

```json
{
  "mcpServers": {
    "local-docs": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "G:\\docs\\malgn"
      ]
    }
  }
}
```

**Step 5: 저장 및 재시작**

1. 저장 (Save)
2. Claude Code 재시작 또는 "Reload" 클릭

**Step 6: 연결 확인**

채팅창에서 테스트:
```
@malgn-docs에서 Page 클래스 사용법을 알려줘
```

또는
```
@company-wiki에서 보안 가이드를 확인해줘
```

### 방법 2: 설정 파일 직접 편집

**Step 1: MCP 설정 파일 위치**

**macOS/Linux:**
```bash
~/.config/claude-code/mcp-servers.json
```

**Windows:**
```
%APPDATA%\claude-code\mcp-servers.json
```

**Step 2: 파일 편집**

**파일: `mcp-servers.json`**

**API 서버 방식:**

```json
{
  "mcpServers": {
    "malgn-docs": {
      "url": "https://mcp.malgnsoft.com/docs",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN"
      }
    },
    "company-wiki": {
      "url": "https://mcp.malgnsoft.com/wiki",
      "headers": {
        "Authorization": "Bearer YOUR_API_TOKEN"
      }
    }
  }
}
```

**파일시스템 방식 (로컬 개발용):**

```json
{
  "mcpServers": {
    "local-docs": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/your/docs"
      ]
    }
  }
}
```

**Step 3: Claude Code 재시작**

## MCP 활용 팁

### 1. 구체적인 문서 위치 명시

**나쁜 예:** "문서 참고해서 만들어줘"

**좋은 예:** "@malgn-docs의 'Page 클래스 사용법'과 @company-wiki의 '템플릿 엔진 가이드'를 참고해서 만들어줘"

### 2. 여러 MCP 서버 조합

여러 문서를 동시에 참조하여 통합 개발:

```
"@malgn-docs에서 프레임워크 패턴을 참고하고,
@company-wiki에서 보안 규칙을 확인하고,
@api-specs에서 API 스펙을 읽어서 인증 모듈을 만들어줘"
```

**효과:** 기술 + 정책 + 계약을 한 번에 적용

### 3. 우선순위 설정

충돌 가능성이 있을 때 우선순위 명시:

```
"@company-wiki의 보안 규칙을 최우선으로 하고,
@malgn-docs의 코딩 패턴을 따라서 만들어줘.
보안 규칙과 충돌하면 반드시 보안 규칙을 따라야 해."
```

### 4. 점진적 학습

단계별로 문서를 참조하여 기능 확장:

```
Step 1: "@malgn-docs에서 기본 CRUD 패턴만 참고해서 구조를 만들어줘"
Step 2: "@company-wiki에서 캐싱 전략을 추가로 참고해서 최적화해줘"
Step 3: "@api-specs에서 검색 API 스펙을 읽고 고급 검색을 추가해줘"
```

### 5. 문서 기반 검증

작성한 코드를 문서 기준으로 검증:

```
"방금 코드를 @malgn-docs와 @company-wiki 기준으로 검증하고,
위반 사항이 있으면 수정해줘."
```

### 6. 기존 코드 학습 및 확장

프로젝트의 기존 패턴을 학습하여 일관성 유지:

```
"@framework-source에서 UserDao를 분석하고,
동일한 패턴으로 ProductDao를 만들어줘."
```

### 7. 질문형 활용

불확실할 때는 먼저 문서 확인 후 작업:

```
"@malgn-docs에서 DataSet.next() 사용법을 먼저 확인해줘.
그 다음 사용자 목록 조회 코드를 작성해줘."
```

## 문제 해결

### MCP 서버가 연결되지 않음

**확인 사항:**
1. Settings에서 MCP 설정 확인
2. JSON 문법 오류 확인 (쉼표, 중괄호)
3. Claude Code 재시작

**문의:**
- 맑은소프트 개발팀에 문의
- MCP 서버 상태 확인 요청

### @mention이 작동하지 않음

**확인:**
1. Claude Code 최신 버전 사용 확인
2. MCP 서버가 정상 연결되었는지 확인
3. Settings > MCP Servers에서 상태 확인

### 문서를 찾지 못함

**원인:**
- MCP 서버에 해당 문서가 없음
- 문서 경로나 이름이 변경됨

**해결:**
- 개발팀에 문서 업데이트 요청
- 다른 MCP 서버 확인

## 보안 고려사항

### Workers 기반 MCP의 보안

맑은소프트의 MCP 서버는 **Cloudflare Workers**로 구축되어 기본적으로 안전합니다:

**자동 보안:**
- ✅ HTTPS 암호화 통신
- ✅ Cloudflare 보안 계층
- ✅ 접근 제어 (인증된 사용자만)
- ✅ 민감 정보 자동 필터링

**사용자 주의사항:**
- MCP 서버 URL을 외부에 공유 금지
- 개인 설정 파일 Git 커밋 금지

## MCP 활용 핵심 요약

### 사용 3원칙

1. **구체적으로** - 문서 위치와 섹션을 명시
2. **조합하여** - 여러 MCP 서버를 함께 활용
3. **검증하여** - 문서 기준으로 코드 리뷰

### 즉시 적용 가능한 패턴

```
# 기본 패턴
"@malgn-docs를 참고해서 {기능}을 만들어줘"

# 고급 패턴
"@malgn-docs와 @company-wiki를 참고하고,
@api-specs의 스펙대로 {기능}을 만들어줘"

# 검증 패턴
"방금 코드를 @malgn-docs 기준으로 검증하고 수정해줘"
```

### 생산성 향상 효과

- ⏱️ **문서 검색 시간**: 10분 → 0분
- 📚 **매뉴얼 숙지**: 1주일 → 즉시
- 🎯 **표준 준수율**: 60% → 95%+
- 🔄 **코드 일관성**: 프로젝트별 차이 → 완전 통일

## 다음 단계

- [프로젝트 구조 표준](project-structure.md) - 표준 폴더 구조
- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트

---

[← 목차로 돌아가기](../_sidebar.md)
