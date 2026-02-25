# 핵심원칙

## 바이브코딩이란

바이브코딩의 본질은 **AI가 원칙을 따라 스스로 개발할 수 있는 최적의 환경을 만드는 것**이다.

코드를 직접 작성하는 것이 아니라, AI가 올바른 코드를 생성할 수 있도록 **규칙, 템플릿, 문서, 명령어**를 체계적으로 설정하는 것이 핵심이다.

## 설정 요소

AI 개발 환경은 다음 6가지 요소로 구성된다:

| 요소 | 위치 | 역할 |
|------|------|------|
| **CLAUDE.md** | 프로젝트 루트 | 프로젝트 개요, 핵심 규칙, 참조 트리거 |
| **rules** | `.claude/rules/` | 자동 적용되는 개발 규칙 (시스템 메시지) |
| **templates** | `.claude/templates/` | 코드 생성 시 참조하는 표준 템플릿 |
| **commands** | `.claude/commands/` | 반복 작업을 위한 슬래시 커맨드 |
| **hooks** | `.claude/hooks/` + `settings.json` | 도구 실행 전후 자동 실행되는 점검 스크립트 |
| **docs** | `docs/` | 상세 개발 문서 (필요 시 참조) |

### 로딩 방식의 차이

- **세션 시작 시 1회**: CLAUDE.md → 세션이 시작될 때 한 번 로딩
- **매 대화마다**: rules → 시스템 메시지에 매번 주입 (최적화 우선순위 가장 높음)
- **요청 시 로딩**: templates, docs → 참조 트리거에 의해 필요할 때만 읽음
- **실행 시 로딩**: commands → 슬래시 커맨드 호출 시 프롬프트로 확장
- **자동 실행**: hooks → 도구 호출 이벤트에 반응하여 실행

## 중복 최적화

### rules는 시스템 메시지다

rules 파일은 **매 대화마다 시스템 메시지에 포함**된다. 따라서:

- **핵심만 작성**: 장황한 설명 대신 규칙과 금지 사항 위주
- **중복 제거**: CLAUDE.md에 이미 있는 내용을 rules에 반복하지 않음
- **상세 내용은 docs로**: 긴 설명이 필요하면 docs/에 작성하고 참조 트리거 사용

```
❌ rules에 넣으면 안 되는 것:
- 튜토리얼이나 예제 코드 나열
- 긴 설명문
- 자주 변경되는 내용

✅ rules에 넣어야 하는 것:
- 필수 패턴 (한 줄 요약)
- 금지 사항
- 참조 트리거 (상세 문서로 유도)
```

### CLAUDE.md vs rules 역할 분담

```
CLAUDE.md:
- 프로젝트 개요 (기술 스택, 구조)
- 핵심 규칙 요약
- 현재 작업 상태
- 주요 참조 트리거

rules:
- 코딩 규칙 (패턴, 금지 사항)
- 스타일 가이드
- 세부 참조 트리거
```

## 참조 트리거

참조 트리거란 AI가 **특정 작업을 할 때 관련 문서를 먼저 읽도록** 유도하는 지시문이다.

### 왜 필요한가

rules에 모든 내용을 넣으면 시스템 메시지가 비대해진다. 대신 핵심 규칙만 rules에 넣고, 상세 내용은 docs/나 templates/에 분리한 뒤 참조 트리거로 연결한다.

### 작성 방법

CLAUDE.md나 rules에서:

```markdown
## 참조 트리거
- 페이지 생성 시 → `.claude/templates/page.md` 참조
- API 엔드포인트 작성 시 → `docs/api.md` 참조
- 폼 처리 시 → `docs/forms.md` 참조
- 인증 관련 작업 시 → `docs/auth.md` 참조
```

commands에서:

```markdown
<!-- .claude/commands/create-page.md -->
페이지를 생성합니다.

## 실행 전 참조
`.claude/templates/page.md`를 먼저 읽고, 해당 패턴에 맞게 코드를 생성하세요.
```

### 트리거 대상

| 대상 | 용도 | 예시 |
|------|------|------|
| templates/ | 코드 생성 패턴 | 페이지, 컴포넌트, 라우트 템플릿 |
| docs/ | 상세 가이드 | API 사용법, 인증, 폼 처리 |
| MCP 서버 | 외부 도구 연동 | DB 스키마 조회, 문서 검색 |

## Hooks

hooks는 Claude Code가 **도구를 실행할 때 자동으로 트리거되는 코딩 점검 스크립트**다.

주로 파일 변경 후 코드 품질을 자동 점검하는 용도로 사용한다. 점검 스크립트를 `.claude/hooks/` 폴더에 작성하고, `.claude/settings.json`에서 해당 스크립트를 실행하도록 설정한다.

### 설정 방법

**1단계: AI에게 점검 스크립트 생성 요청**

```
프롬프트: "이 프로젝트의 코딩 규칙에 맞는 hooks 점검 스크립트를
.claude/hooks/post-edit.sh에 만들어줘.
파일 변경 후 자동으로 코딩 규칙 위반을 점검하도록 해줘."
```

AI가 프로젝트의 rules와 CLAUDE.md를 참고하여 해당 프로젝트에 맞는 점검 스크립트를 자동 생성한다.

**2단계: settings.json에 hooks 등록**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "bash .claude/hooks/post-edit.sh"
      }
    ]
  }
}
```

### 구조

```
.claude/
├── hooks/
│   └── post-edit.sh        # 파일 변경 후 점검 스크립트
├── settings.json            # hooks 실행 설정
├── rules/
├── commands/
└── templates/
```

### 동작 원리

- 점검 스크립트가 `exit 1`을 반환하면 Claude Code에 경고가 전달되어, AI가 문제를 인지하고 수정한다
- 프로젝트 규칙이 변경되면 AI에게 스크립트 업데이트를 요청하여 점검 항목을 최신 상태로 유지한다

## 설정 최적화 점검

프로젝트 설정이 완료되면, AI에게 직접 점검을 요청한다:

```
프롬프트 예시:
"CLAUDE.md와 .claude/ 폴더의 설정을 검토해줘.
- rules에 불필요하게 긴 내용이 있는지
- 중복되는 내용이 있는지
- 빠진 참조 트리거가 있는지
- templates/commands가 실제 개발 흐름에 맞는지
개선 사항을 제안해줘."
```

이 점검을 통해:
- rules의 시스템 메시지 크기를 최소화
- 누락된 참조 트리거를 보완
- commands와 templates의 실용성을 검증

## 체크리스트

AI 개발 환경 구축 확인:

- [ ] CLAUDE.md에 프로젝트 개요와 핵심 규칙이 있는가?
- [ ] rules에 핵심만 들어가 있는가? (장황한 설명 없이)
- [ ] CLAUDE.md와 rules 사이에 내용 중복이 없는가?
- [ ] 상세 내용이 docs/로 분리되어 있는가?
- [ ] templates/에 코드 생성 패턴이 있는가?
- [ ] commands/에 반복 작업 커맨드가 있는가?
- [ ] 참조 트리거가 적절히 설정되어 있는가?
- [ ] AI에게 설정 최적화 점검을 요청했는가?

> 바이브코딩의 핵심은 코드를 작성하는 것이 아니라, **AI가 스스로 올바른 코드를 만들어내는 환경을 설계하는 것**이다.

## 관련 문서

- [CLAUDE.md 작성](claude-md.md)
- [.claude/rules 작성](claude-rules.md)
- [.claude/commands 활용](claude-commands.md)
- [.claude/templates 활용](claude-templates.md)
