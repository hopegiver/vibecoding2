# .claude/hooks 활용 가이드

## 왜 hooks가 바이브코딩의 핵심인가?

바이브코딩에서 AI는 코드를 빠르게 생성하지만, **생성된 코드가 올바른지 검증하는 비용**이 문제입니다.

Hook 없이 AI가 오류를 발견하는 과정:

```
코드 작성 → AI가 "확인해볼게요" → 파일 읽기 → 린트 실행 → 결과 읽기 → 수정
```

이 과정에서 **5~6번의 도구 호출과 수천 토큰**이 소비됩니다. 세션이 길어질수록 이런 검증 비용이 누적되어 컨텍스트를 압박합니다.

Hook을 사용하면:

```
코드 작성 → Hook이 자동 검증 → 오류 시 AI에게 즉시 피드백 → 수정
```

**1번의 자동 실행으로 끝납니다.** 토큰도, 시간도, 실수도 줄어듭니다.

### rules vs hooks

| | rules | hooks |
|--|-------|-------|
| 방식 | "하지 마라"고 **말하는** 것 | 하면 **즉시 잡아주는** 것 |
| AI가 어기면? | 아무 일도 안 생김 | 에러가 반환되어 AI가 즉시 수정 |
| 비유 | 교통 법규 | 과속 카메라 |

**rules는 가이드라인이고, hooks는 실제 방어선입니다.** 둘 다 필요하지만, 확실하게 막아야 하는 것은 hooks로 구현하세요.

### hooks의 3가지 효과

**1. 토큰 절약** — AI가 스스로 검증하는 과정(파일 읽기 → 린트 → 결과 확인)을 Hook 한 줄이 대체합니다. 세션이 길어질수록 절약 효과가 커집니다.

**2. 즉시 피드백** — AI가 코드를 저장하는 순간 오류를 감지합니다. 여러 파일을 수정한 후에야 문제를 발견하는 것보다 훨씬 효율적입니다.

**3. 실수 방지** — rules는 세션이 길어지면 AI가 잊을 수 있지만, hooks는 기계적으로 실행되므로 **절대 잊지 않습니다.**

## hooks 기본 구조

`.claude/settings.json`에 설정합니다:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
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

### hook 이벤트

| 이벤트 | 실행 시점 | 주요 용도 |
|--------|----------|----------|
| `PostToolUse` | 도구 실행 **후** | 파일 수정 후 검증 |
| `PreToolUse` | 도구 실행 **전** | 위험한 작업 사전 차단 |
| `Stop` | AI 응답 완료 시 | 최종 검증 |

**가장 중요한 이벤트는 `PostToolUse`입니다.** 파일이 수정될 때마다 자동 검증을 실행합니다.

### matcher 패턴

```json
"matcher": "Write|Edit"
```

| 도구명 | 설명 |
|--------|------|
| `Edit` | 파일 부분 수정 |
| `Write` | 파일 전체 작성 |
| `Bash` | 셸 명령 실행 |

### 종료 코드

| 종료 코드 | 의미 |
|----------|------|
| `0` | 성공 — AI가 계속 진행 |
| `1` | 오류 — **stdout 내용을 AI가 읽고 자동 수정** |
| `2` | 차단 — AI 실행 강제 중단 |

핵심은 **`exit 1` + 오류 메시지**입니다. AI가 메시지를 읽고 스스로 수정을 시도합니다.

## 실전 활용 패턴

### 패턴 1: 자동 등록 (vue-zero-template)

vue-zero-template에 포함된 Hook입니다. AI가 파일을 만들면 등록 파일을 자동 갱신합니다:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "node bin/hook-scan.js || true"
          }
        ]
      }
    ]
  }
}
```

**효과:**
- `.vue` 파일 생성 → `pages.json`, `components.json` 자동 갱신
- `server/api/*.js` 생성 → `_registry.js` 자동 갱신
- AI가 "등록 파일도 수정해야지"라고 기억할 필요 없음

### 패턴 2: 규칙 위반 즉시 감지

AI가 금지된 패턴을 사용하면 즉시 잡아냅니다:

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | node -e "
  let d=''; process.stdin.on('data',c=>d+=c);
  process.stdin.on('end',()=>{
    const o=JSON.parse(d).tool_input||{};
    console.log(o.file_path||o.path||'');
  })
" 2>/dev/null)

[ -z "$FILE" ] || [ ! -f "$FILE" ] && exit 0

# Vue Zero: <style> 태그 금지
if echo "$FILE" | grep -qE '\.vue$'; then
  if grep -q '<style' "$FILE"; then
    echo "규칙 위반: .vue 파일에 <style> 태그 사용 금지. Bootstrap 클래스를 사용하세요."
    exit 1
  fi
fi

# 맑은프레임워크: JSP에 HTML 금지
if echo "$FILE" | grep -q '\.jsp$'; then
  if grep -qE '<html>|<body>|<div' "$FILE"; then
    echo "규칙 위반: JSP 파일에 HTML 직접 작성 금지. HTML은 별도 템플릿 파일로 분리하세요."
    exit 1
  fi
fi

exit 0
```

**효과:** rules에 "금지"라고 써놓은 것을 hooks가 실제로 강제합니다.

### 패턴 3: 문법 오류 즉시 감지

```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | node -e "
  let d=''; process.stdin.on('data',c=>d+=c);
  process.stdin.on('end',()=>{
    const o=JSON.parse(d).tool_input||{};
    console.log(o.file_path||o.path||'');
  })
" 2>/dev/null)

[ -z "$FILE" ] || [ ! -f "$FILE" ] && exit 0

# JavaScript 문법 체크
if echo "$FILE" | grep -qE '\.(js|mjs)$'; then
  RESULT=$(node --check "$FILE" 2>&1)
  if [ $? -ne 0 ]; then
    echo "문법 오류: $FILE"
    echo "$RESULT"
    exit 1
  fi
fi

exit 0
```

**효과:** AI가 문법 오류가 있는 코드를 작성하면 즉시 감지하여 수정하게 합니다.

## hooks 설계 원칙

### 빠르게 실행되어야 합니다

Hook은 AI가 파일을 수정할 **때마다** 실행됩니다. 느리면 전체 작업 속도가 떨어집니다.

✅ **좋은 예:**
- `node --check file.js` (100ms 이내)
- `grep` 패턴 매칭 (즉시)
- 단일 파일 린트 (1초 이내)

❌ **나쁜 예:**
- 프로젝트 전체 빌드 (수십 초)
- 전체 테스트 스위트 실행 (수 분)
- 외부 API 호출 (네트워크 지연)

### 오류 메시지는 구체적으로

AI가 오류 메시지를 읽고 수정하므로, **무엇이 잘못되었고 어떻게 고쳐야 하는지** 알려줘야 합니다.

✅ **좋은 메시지:**
```
규칙 위반: .vue 파일에 <style> 태그 사용 금지. Bootstrap 클래스를 사용하세요.
```

❌ **나쁜 메시지:**
```
Error
```

### 실패해도 안전하게

Hook 자체의 오류로 AI 작업이 중단되면 안 됩니다:

```json
"command": "node bin/hook-scan.js || true"
```

`|| true`를 붙이면 스크립트가 실패해도 AI 작업이 계속 진행됩니다. 검증용 Hook이 아닌 자동화 Hook에 사용합니다.

## 디버깅

### 수동 테스트

```bash
echo '{"tool_name":"Edit","tool_input":{"file_path":"app/pages/index.vue"}}' | node bin/hook-scan.js
```

### 로그 확인

```bash
# 스크립트 상단에 추가
exec >> /tmp/claude-hooks.log 2>&1
echo "=== $(date) === FILE: $FILE"
```

```bash
# 실시간 로그 확인
tail -f /tmp/claude-hooks.log
```

## 관련 문서

- [.claude/rules 작성](claude-rules.md) — 규칙 기반 가이드라인
- [.claude/skills 활용](claude-skills.md) — 재사용 가능한 워크플로우
- [CLAUDE.md 작성](claude-md.md) — 프로젝트 맥락 정보

---

[← 목차로 돌아가기](../_sidebar.md)
