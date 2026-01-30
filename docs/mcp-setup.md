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

```json
{
  "mcpServers": {
    "malgn-docs": {
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
@malgn-docs에서 문서 목록을 보여줘
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

```json
{
  "mcpServers": {
    "filesystem-docs": {
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

## 실전 예제

### 예제 1: 맑은프레임워크 문서 연동

회사에 맑은프레임워크 매뉴얼이 `G:\docs\malgn\` 폴더에 있다고 가정합니다.

**설정:**

```json
{
  "mcpServers": {
    "malgn-docs": {
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

**사용:**

```
프롬프트: "@malgn-docs에서 Page 클래스의 setLoop 메소드 사용법을 찾아서
사용자 목록 페이지를 만들어줘"
```

**Claude Code 동작:**
1. `G:\docs\malgn\` 폴더에서 `setLoop` 관련 문서 검색
2. 정확한 사용법 확인
3. 맑은프레임워크 규칙을 따르는 코드 생성

### 예제 2: Confluence 위키 연동

사내 Confluence에 개발 가이드가 있는 경우:

**설정:**

```json
{
  "mcpServers": {
    "company-wiki": {
      "command": "node",
      "args": [
        "/path/to/confluence-mcp-server.js"
      ],
      "env": {
        "CONFLUENCE_URL": "https://wiki.company.com",
        "CONFLUENCE_TOKEN": "your-api-token"
      }
    }
  }
}
```

**사용:**

```
프롬프트: "@company-wiki에서 JWT 인증 가이드를 찾아서 로그인 API를 만들어줘"
```

### 예제 3: GitHub 저장소 연동

프레임워크 소스 코드를 참조:

```json
{
  "mcpServers": {
    "framework-source": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github",
        "--repo",
        "company/malgn-framework"
      ],
      "env": {
        "GITHUB_TOKEN": "your-github-token"
      }
    }
  }
}
```

**사용:**

```
프롬프트: "@framework-source에서 DataSet 클래스 구현을 참고해서
커스텀 ResultSet 클래스를 만들어줘"
```

## MCP 서버 구축

맑은소프트에서는 **Cloudflare Workers 기반 MCP 서버**를 구축하여 사용합니다.

**구축 담당:** 맑은소프트 개발팀
**사용 방법:** 개발팀이 제공하는 MCP 서버 URL을 설정에 추가

구체적인 구축 방법은 별도 문서 참조.

## MCP 활용 패턴

### 패턴 1: 문서 우선 개발

**사내 표준 자동 준수**

```
프롬프트: "@malgn-docs와 @company-wiki를 참고해서
맑은프레임워크 기반 사용자 관리 모듈을 만들어줘.

요구사항:
- 사내 코딩 표준 준수
- 맑은프레임워크 패턴 사용
- 보안 가이드 적용

파일:
- public_html/user/user_list.jsp
- public_html/user/user_form.jsp
- src/dao/UserDao.java
```

**효과:**
- ✅ 매뉴얼 찾아보는 시간 절약
- ✅ 표준 패턴 자동 적용
- ✅ 일관된 코드 품질

### 패턴 2: 기존 코드 학습 및 확장

**기존 패턴 분석 후 복제**

```
프롬프트: "@framework-source에서 기존 UserDao 클래스를 분석하고,
동일한 패턴으로 ProductDao 클래스를 만들어줘.

포함 기능:
- findAll (페이징)
- findById
- insert
- update
- delete
- search (제품명, 카테고리)
```

**효과:**
- ✅ 프로젝트 코딩 스타일 자동 유지
- ✅ 검증된 패턴 재사용
- ✅ 신규 개발자도 일관된 코드 작성

### 패턴 3: API 스펙 기반 개발

**OpenAPI/Swagger 스펙 자동 구현**

```
프롬프트: "@api-specs의 /api/products OpenAPI 스펙을 읽고,
모든 엔드포인트를 구현해줘.

Workers 구조:
- src/routes/products.js
- src/services/productService.js
- D1 데이터베이스 사용
- JWT 인증 적용
```

**효과:**
- ✅ 스펙과 구현 자동 일치
- ✅ 빠뜨린 엔드포인트 없음
- ✅ 프론트엔드와 즉시 연동 가능

### 패턴 4: 다중 문서 통합 참조

**여러 문서를 동시에 활용**

```
프롬프트: "@malgn-docs의 DataSet 사용법과
@company-wiki의 보안 가이드를 참고하여
게시판 목록 조회 기능을 만들어줘.

보안 요구사항:
- XSS 필터링 (m.rs 사용)
- SQL Injection 방지 (PreparedStatement)
- 권한 체크 (Auth.isLogin)

기능:
- 페이징 (ListManager)
- 검색 (제목+내용)
- 정렬 (최신순, 조회수순)
```

**효과:**
- ✅ 여러 표준을 동시에 적용
- ✅ 보안과 기능 모두 충족
- ✅ 복잡한 요구사항 한 번에 해결

### 패턴 5: 레거시 마이그레이션

**기존 시스템 분석 후 현대화**

```
프롬프트: "@legacy-code에서 기존 JSP 직접 SQL 코드를 분석하고,
@malgn-docs의 DAO 패턴으로 리팩토링해줘.

기존: board_list.jsp (JSP에서 직접 SQL)
신규:
- src/dao/BoardDao.java (DAO 패턴)
- board_list.jsp (깔끔한 JSP, HTML 템플릿 분리)
```

**효과:**
- ✅ 레거시 코드 자동 분석
- ✅ 현대 패턴으로 안전하게 전환
- ✅ 실수 없이 마이그레이션

### 패턴 6: 실시간 문서 동기화

**최신 프레임워크 버전 자동 반영**

```
프롬프트: "@malgn-docs에서 최신 1.15.0 버전의 새 기능을 확인하고,
기존 코드에 적용 가능한 부분을 알려줘.

체크 항목:
- 성능 개선 기능
- 보안 강화 기능
- Deprecated 메소드
```

**효과:**
- ✅ 최신 기능 자동 인지
- ✅ 기술 부채 방지
- ✅ 업그레이드 계획 자동 수립

## MCP 활용 극대화 팁

### 1. 구체적인 문서 위치 명시

**❌ 나쁜 예:**
```
"문서 참고해서 만들어줘"
```

**✅ 좋은 예:**
```
"@malgn-docs의 'Page 클래스 사용법' 섹션과
@company-wiki의 '템플릿 엔진 가이드'를 참고해서 만들어줘"
```

### 2. 여러 MCP 서버 조합

**시너지 효과 극대화:**

```
프롬프트:
"@malgn-docs에서 프레임워크 패턴을 참고하고,
@company-wiki에서 사내 보안 규칙을 확인하고,
@api-specs에서 API 계약을 읽어서
완전한 인증 모듈을 만들어줘"
```

**3가지 정보원을 통합:**
- 프레임워크 패턴 (기술)
- 사내 보안 규칙 (정책)
- API 스펙 (계약)

### 3. 컨텍스트 우선순위 설정

**명확한 우선순위 지정:**

```
프롬프트:
"@company-wiki의 보안 규칙을 최우선으로 하고,
@malgn-docs의 코딩 패턴을 따라서 만들어줘.

단, 보안 규칙과 프레임워크 패턴이 충돌하면
반드시 보안 규칙을 따라야 해."
```

### 4. 점진적 학습 활용

**단계별 문서 참조:**

```
Step 1:
"@malgn-docs에서 기본 CRUD 패턴만 참고해서 기본 구조를 만들어줘"

Step 2:
"@company-wiki에서 캐싱 전략을 추가로 참고해서 성능 최적화해줘"

Step 3:
"@api-specs에서 검색 API 스펙을 읽고 고급 검색 기능을 추가해줘"
```

### 5. 질문형 활용

**문서 내용 확인 후 작업:**

```
프롬프트:
"@malgn-docs에서 DataSet.next()를 어떻게 사용하는지 먼저 확인해줘.
그 다음 사용자 목록 조회 코드를 작성해줘."
```

**2단계 프로세스:**
1. 문서에서 정확한 사용법 확인
2. 확인한 내용으로 코드 작성

### 6. 네거티브 프롬프트

**하지 말아야 할 것 명시:**

```
프롬프트:
"@company-wiki의 금지 패턴 목록을 확인하고,
절대 사용하지 말고 사용자 등록 API를 만들어줘.

특히:
- try-catch 사용 금지
- JSP에 HTML 직접 작성 금지
- SQL Injection 취약 패턴 금지"
```

### 7. 검증 요청

**문서 기준으로 코드 리뷰:**

```
프롬프트:
"방금 작성한 코드를 @malgn-docs와 @company-wiki를 기준으로
검증해줘.

체크 항목:
- 프레임워크 패턴 준수 여부
- 보안 규칙 준수 여부
- 성능 최적화 적용 여부

위반 사항이 있으면 즉시 수정해줘."
```

### 8. 버전별 문서 활용

**특정 버전 명시:**

```
프롬프트:
"@malgn-docs-v1.14에서 구버전 DataSet 사용법을 확인하고,
@malgn-docs-v1.15에서 신버전 DataSet 사용법을 비교해줘.

그리고 마이그레이션 가이드를 작성해줘."
```

### 9. 실시간 업데이트 활용

**최신 정보 자동 반영:**

```
프롬프트:
"@company-wiki에서 오늘 업데이트된 보안 패치 내용을 확인하고,
기존 인증 코드에 즉시 적용해줘."
```

### 10. 템플릿 생성 요청

**문서 기반 재사용 템플릿:**

```
프롬프트:
"@malgn-docs의 CRUD 패턴을 분석해서
앞으로 재사용할 수 있는 DAO 템플릿을 templates/DaoTemplate.java로
만들어줘.

포함 사항:
- 기본 CRUD 메소드
- 페이징
- 검색
- 정렬
- 주석 (JavaDoc)"
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
