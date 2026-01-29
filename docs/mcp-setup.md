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

### Step 1: MCP 설정 파일 위치

**macOS/Linux:**
```bash
~/.config/claude-code/mcp-servers.json
```

**Windows:**
```
%APPDATA%\claude-code\mcp-servers.json
```

### Step 2: 기본 설정 예제

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

## MCP 서버 만들기 (고급)

자체 MCP 서버를 만들 수도 있습니다.

### 간단한 MCP 서버 예제

**파일: `custom-mcp-server.js`**

```javascript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new Server(
  {
    name: 'custom-docs-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      resources: {},
    },
  }
);

// 리소스 목록 제공
server.setRequestHandler('resources/list', async () => {
  return {
    resources: [
      {
        uri: 'doc://coding-guide',
        name: '코딩 가이드',
        mimeType: 'text/markdown',
      },
    ],
  };
});

// 리소스 내용 제공
server.setRequestHandler('resources/read', async (request) => {
  if (request.params.uri === 'doc://coding-guide') {
    return {
      contents: [
        {
          uri: 'doc://coding-guide',
          mimeType: 'text/markdown',
          text: '# 코딩 가이드\n\n...',
        },
      ],
    };
  }
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

**설정:**

```json
{
  "mcpServers": {
    "custom-docs": {
      "command": "node",
      "args": ["custom-mcp-server.js"]
    }
  }
}
```

## MCP 활용 패턴

### 패턴 1: 문서 우선 개발

```
프롬프트: "@malgn-docs와 @company-wiki를 참고해서
맑은프레임워크 기반 사용자 관리 모듈을 만들어줘.

요구사항:
- 사내 코딩 표준 준수
- 맑은프레임워크 패턴 사용
- 보안 가이드 적용"
```

### 패턴 2: 기존 코드 학습

```
프롬프트: "@framework-source에서 기존 UserDao 클래스를 분석하고,
같은 패턴으로 ProductDao 클래스를 만들어줘"
```

### 패턴 3: API 스펙 기반 개발

```
프롬프트: "@api-specs의 OpenAPI 스펙을 읽고,
모든 엔드포인트를 구현해줘"
```

## 문제 해결

### MCP 서버가 연결되지 않음

**확인 사항:**
1. `mcp-servers.json` 파일 경로 확인
2. JSON 문법 오류 확인 (쉼표, 중괄호)
3. `command` 경로가 올바른지 확인

**디버깅:**
```bash
# MCP 서버 직접 실행 테스트
npx @modelcontextprotocol/server-filesystem /path/to/docs
```

### 문서를 찾지 못함

**확인 사항:**
1. 문서 경로가 올바른지 확인
2. 파일 권한 확인 (읽기 권한 필요)
3. 문서 형식 확인 (지원: .md, .txt, .pdf)

### 프롬프트에 @가 작동하지 않음

Claude Code 버전을 확인하세요. MCP는 최신 버전에서만 지원됩니다.

```bash
# VSCode에서 확장 업데이트 확인
Ctrl+Shift+X → "Claude Code" → Update
```

## 보안 고려사항

### 1. API 토큰 관리

환경 변수 사용:

```json
{
  "mcpServers": {
    "secure-wiki": {
      "command": "node",
      "args": ["wiki-server.js"],
      "env": {
        "WIKI_TOKEN": "${WIKI_API_TOKEN}"
      }
    }
  }
}
```

시스템 환경 변수에 `WIKI_API_TOKEN` 설정.

### 2. 접근 제한

MCP 서버에 접근 가능한 경로를 제한:

```json
{
  "mcpServers": {
    "safe-docs": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/safe/docs/only"
      ]
    }
  }
}
```

### 3. 민감 정보 제외

`.gitignore`, `.env` 등 민감 정보는 MCP에서 제외:

```javascript
// MCP 서버에서 필터링
if (filePath.includes('.env') || filePath.includes('secrets')) {
  return null; // 접근 거부
}
```

## 다음 단계

- [프로젝트 구조 표준](project-structure.md) - 표준 폴더 구조
- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트

## 참고 자료

- [MCP 공식 문서](https://modelcontextprotocol.io/)
- [MCP SDK GitHub](https://github.com/modelcontextprotocol/sdk)
- [MCP 서버 예제](https://github.com/modelcontextprotocol/servers)

---

[← 목차로 돌아가기](../_sidebar.md)
