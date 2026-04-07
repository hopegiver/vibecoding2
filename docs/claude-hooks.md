# .claude/hooks 활용 가이드

## 왜 hooks가 필요한가?

Claude Code는 코드를 작성하지만, 작성된 코드가 실제로 올바른지는 **기계적으로 검증**해야 합니다. `hooks`는 Claude의 특정 행동 전후에 셸 스크립트를 자동 실행하는 기능입니다.

핵심 활용 사례: **코딩 후 자동 린트/타입체크/테스트**로 오류를 즉시 탐지합니다.

---

## hooks 기본 구조

`settings.json`에 `hooks` 섹션을 추가합니다.

```json
{
  "permissions": { ... },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "스크립트 경로 또는 명령어"
          }
        ]
      }
    ]
  }
}
```

---

## hook 이벤트 종류

| 이벤트 | 실행 시점 |
|--------|----------|
| `PreToolUse` | 도구 실행 **전** |
| `PostToolUse` | 도구 실행 **후** |
| `Notification` | Claude가 알림을 보낼 때 |
| `Stop` | Claude가 응답을 완료할 때 |
| `SubagentStop` | 서브에이전트가 완료될 때 |

가장 중요한 이벤트: **`PostToolUse`** — 파일 수정 직후 검증 스크립트 실행에 사용합니다.

---

## matcher 패턴

```json
"matcher": "Edit|Write|MultiEdit"
```

파이프(`|`)로 여러 도구를 OR 조건으로 지정합니다.

| 도구명 | 설명 |
|--------|------|
| `Edit` | 파일 부분 수정 |
| `Write` | 파일 전체 작성 |
| `MultiEdit` | 여러 위치 동시 수정 |
| `Bash` | 셸 명령 실행 |

---

## 실전 설정: 코딩 후 자동 오류 탐지

### 1. settings.json 전체 예시

```json
{
  "permissions": {
    "allow": ["Read", "Edit", "Write", "Glob", "Grep", "Bash"],
    "deny": ["Bash(rm -rf *)"]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/post-edit-check.sh"
          }
        ]
      }
    ]
  }
}
```

### 2. 오류 탐지 스크립트 작성

`.claude/hooks/post-edit-check.sh`

```bash
#!/bin/bash

# hook 입력: stdin으로 JSON 수신
INPUT=$(cat)

# 수정된 파일 경로 추출
TOOL=$(echo "$INPUT" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('tool_name',''))" 2>/dev/null)
FILE=$(echo "$INPUT" | python3 -c "import sys,json; d=json.load(sys.stdin); inp=d.get('tool_input',{}); print(inp.get('file_path', inp.get('path','')))" 2>/dev/null)

# 파일이 없으면 종료
[ -z "$FILE" ] && exit 0
[ ! -f "$FILE" ] && exit 0

ERRORS=""

# JavaScript/TypeScript 검사
if echo "$FILE" | grep -qE '\.(js|ts|jsx|tsx|mjs)$'; then
  # ESLint
  if command -v npx &>/dev/null && [ -f ".eslintrc*" -o -f "eslint.config*" ]; then
    RESULT=$(npx eslint --no-eslintrc --rule '{"no-undef":"error","no-unused-vars":"warn"}' "$FILE" 2>&1)
    [ $? -ne 0 ] && ERRORS+="[ESLint]\n$RESULT\n"
  fi

  # TypeScript 타입 체크
  if echo "$FILE" | grep -qE '\.(ts|tsx)$'; then
    if command -v npx &>/dev/null && [ -f "tsconfig.json" ]; then
      RESULT=$(npx tsc --noEmit --skipLibCheck 2>&1 | head -20)
      [ $? -ne 0 ] && ERRORS+="[TypeScript]\n$RESULT\n"
    fi
  fi
fi

# Python 검사
if echo "$FILE" | grep -q '\.py$'; then
  # 문법 체크
  RESULT=$(python3 -m py_compile "$FILE" 2>&1)
  [ $? -ne 0 ] && ERRORS+="[Python 문법]\n$RESULT\n"

  # flake8 (설치된 경우)
  if command -v flake8 &>/dev/null; then
    RESULT=$(flake8 --max-line-length=120 "$FILE" 2>&1)
    [ $? -ne 0 ] && ERRORS+="[flake8]\n$RESULT\n"
  fi
fi

# Java 검사 (간단한 문법 검사)
if echo "$FILE" | grep -q '\.java$'; then
  if command -v javac &>/dev/null; then
    RESULT=$(javac -cp ".:src/main/java" "$FILE" 2>&1)
    [ $? -ne 0 ] && ERRORS+="[javac]\n$RESULT\n"
  fi
fi

# HTML 검사
if echo "$FILE" | grep -q '\.html$'; then
  # <script> 태그 금지 체크 (ViewLogic 규칙)
  if grep -q '<script' "$FILE"; then
    ERRORS+="[ViewLogic 규칙 위반] HTML 파일에 <script> 태그 사용 금지: $FILE\n"
  fi
  if grep -q '<style' "$FILE"; then
    ERRORS+="[ViewLogic 규칙 위반] HTML 파일에 <style> 태그 사용 금지: $FILE\n"
  fi
fi

# 오류 출력
if [ -n "$ERRORS" ]; then
  echo "::error 코드 검사 실패"
  printf "$ERRORS"
  exit 1
fi

exit 0
```

---

## hook 입출력 규약

Claude는 hook 스크립트에 **stdin**으로 JSON을 전달합니다.

```json
{
  "session_id": "abc123",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/path/to/file.ts",
    "old_string": "...",
    "new_string": "..."
  }
}
```

### 스크립트 종료 코드 의미

| 종료 코드 | 의미 |
|----------|------|
| `0` | 성공 — Claude가 계속 진행 |
| `1` | 오류 — stdout 내용을 Claude가 읽고 자동 수정 시도 |
| `2` | 차단 — Claude 실행 강제 중단 |

`exit 1` + 오류 메시지 출력 → Claude가 오류를 읽고 **자동으로 수정을 시도**합니다.

---

## 플랫폼별 hooks 설정

### Cloudflare Workers (Hono)

```bash
# .claude/hooks/post-edit-check.sh 추가 내용

# Workers: Wrangler 타입 체크
if [ -f "wrangler.toml" ] && echo "$FILE" | grep -qE '\.(ts)$'; then
  RESULT=$(npx tsc --noEmit 2>&1 | head -20)
  [ $? -ne 0 ] && ERRORS+="[Workers TypeScript]\n$RESULT\n"
fi
```

### 맑은프레임워크 (Java/JSP)

```bash
# 맑은프레임워크 규칙 위반 탐지
if echo "$FILE" | grep -q '\.java$'; then
  # try-catch 사용 금지 탐지
  if grep -qE 'try\s*\{|catch\s*\(' "$FILE"; then
    ERRORS+="[맑은프레임워크 규칙] try-catch 사용 금지. boolean 리턴으로 오류 처리하세요: $FILE\n"
  fi
fi

if echo "$FILE" | grep -q '\.jsp$'; then
  # JSP에 HTML 직접 작성 금지
  if grep -qE '<html>|<body>|<div' "$FILE"; then
    ERRORS+="[맑은프레임워크 규칙] JSP 파일에 HTML 직접 작성 금지: $FILE\n"
  fi
fi
```

---

## hooks 디버깅

### 로그 파일로 확인

```bash
# .claude/hooks/post-edit-check.sh 맨 위에 추가
exec >> /tmp/claude-hooks.log 2>&1
echo "=== $(date) ==="
echo "FILE: $FILE"
```

```bash
# 로그 실시간 확인
tail -f /tmp/claude-hooks.log
```

### 수동 테스트

```bash
# hook 직접 실행 테스트
echo '{"tool_name":"Edit","tool_input":{"file_path":"src/index.ts"}}' | bash .claude/hooks/post-edit-check.sh
```

---

## 최소 설정 (빠른 시작)

프로젝트에 바로 적용할 수 있는 최소 구성입니다.

**`.claude/settings.json`:**

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write|MultiEdit",
        "hooks": [
          {
            "type": "command",
            "command": "bash .claude/hooks/post-edit-check.sh"
          }
        ]
      }
    ]
  }
}
```

**`.claude/hooks/post-edit-check.sh`:**

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | python3 -c "
import sys, json
d = json.load(sys.stdin)
inp = d.get('tool_input', {})
print(inp.get('file_path', inp.get('path', '')))
" 2>/dev/null)

[ -z "$FILE" ] || [ ! -f "$FILE" ] && exit 0

# JS/TS 문법 체크 (node 설치 시)
if echo "$FILE" | grep -qE '\.(js|ts|jsx|tsx)$'; then
  if command -v node &>/dev/null; then
    node --check "$FILE" 2>&1 && exit 0
    echo "문법 오류가 발견되었습니다: $FILE"
    node --check "$FILE" 2>&1
    exit 1
  fi
fi

exit 0
```

---

## 체크리스트

- [ ] `.claude/hooks/` 폴더 생성
- [ ] `post-edit-check.sh` 작성 및 실행 권한 부여 (`chmod +x`)
- [ ] `settings.json`에 `hooks` 섹션 추가
- [ ] 플랫폼별 규칙 탐지 로직 추가 (ViewLogic, 맑은프레임워크 등)
- [ ] `/tmp/claude-hooks.log`로 정상 동작 확인

---

## 관련 문서

- [.claude/rules 작성](claude-rules.md) — 규칙 기반 코드 작성 가이드
- [.claude/skills 활용](claude-skills.md) — 재사용 가능한 워크플로우
- [핵심원칙](core-principles.md) — 바이브코딩 핵심 원칙

[← 목차로 돌아가기](../_sidebar.md)
