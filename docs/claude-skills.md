# .claude/skills 활용 가이드

## 스킬이란?

`.claude/skills/`는 반복적인 작업을 슬래시 명령어(`/스킬명`)로 실행하거나, Claude가 상황에 따라 자동 호출하게 만드는 기능입니다.

> 기존 `.claude/commands/`의 상위 호환입니다. Commands도 계속 작동합니다.

## 언제 스킬이 필요한가?

AI가 똑똑해지면서 단순한 코드 생성은 자연어 요청만으로 충분해졌습니다. **스킬이 여전히 가치 있는 경우**는 다음과 같습니다:

### 스킬이 필요한 경우

| 상황 | 이유 |
|------|------|
| **순서가 중요한 멀티스텝 작업** | DB 마이그레이션 → 스키마 변경 → DAO 수정 → API 수정 → 테스트 같은 흐름을 AI가 매번 같은 순서로 실행하게 보장 |
| **회사 고유 프로세스** | 배포 절차, 코드 리뷰 체크리스트 등 AI가 맥락을 모르는 사내 규칙 |
| **위험한 작업의 체크리스트** | 배포, 데이터 삭제 등 실수하면 안 되는 작업의 단계별 검증 |

### 스킬이 불필요한 경우

| 상황 | 대안 |
|------|------|
| 단순 코드 생성 (CRUD, 페이지, 컴포넌트) | 자연어 요청으로 충분 |
| 코딩 규칙 강제 | rules + hooks로 해결 |
| 코드 패턴 일관성 | CLAUDE.md + templates로 해결 |

**판단 기준:** "AI에게 자연어로 요청하면 매번 같은 결과가 나오는가?" → Yes면 스킬 불필요, No면 스킬로 절차를 고정.

## 기본 구조

```
.claude/skills/
├── deploy/
│   └── SKILL.md
└── review/
    └── SKILL.md
```

### SKILL.md 작성법

```markdown
---
name: deploy
description: 프로덕션 배포 체크리스트를 실행합니다.
disable-model-invocation: true
---

스킬 지시사항을 여기에 작성합니다.
$ARGUMENTS 위치에 사용자가 전달한 인자가 들어옵니다.
```

### 주요 Frontmatter 필드

| 필드 | 설명 | 기본값 |
|------|------|-------|
| `name` | 슬래시 명령어 이름 | 폴더명 |
| `description` | 자동 트리거 판단 기준 | 첫 단락 |
| `disable-model-invocation` | `true`면 사용자만 실행 가능 | `false` |
| `context` | `fork`면 별도 subagent에서 실행 | 인라인 |
| `allowed-tools` | 이 스킬에서 허용할 도구 | 세션 설정 |

## 실전 예시

### 배포 체크리스트 (수동 전용)

배포처럼 실수하면 안 되는 작업은 스킬로 절차를 고정합니다. `/deploy production`을 실행하면 AI가 다음 순서대로 자동 실행합니다:

```markdown
---
name: deploy
description: 프로덕션 배포 체크리스트를 실행합니다.
disable-model-invocation: true
allowed-tools: Bash Read
---

# 배포 체크리스트

현재 git 상태: !`git status --short`
최근 커밋: !`git log --oneline -5`

## 배포 대상: $ARGUMENTS

## AI가 순서대로 실행할 단계

1. 테스트 통과 확인 (`npm run test`)
2. 환경 변수 점검 (시크릿이 프로덕션에 등록되었는지)
3. `npm run scan`으로 등록 파일 최신화
4. `wrangler deploy --env $ARGUMENTS` 실행
5. 프로덕션 URL 헬스체크 확인

각 단계의 결과를 보고하고, 실패 시 원인을 분석하세요.
```

### 코드베이스 조사 (독립 실행)

메인 컨텍스트를 오염시키지 않고 독립적으로 조사합니다:

```markdown
---
name: research
description: 코드베이스의 특정 주제를 독립적으로 조사합니다.
context: fork
agent: Explore
---

$ARGUMENTS에 대해 코드베이스 전체를 분석합니다.

1. 관련 파일 탐색
2. 코드 패턴 분석
3. 파일 경로와 줄 번호를 포함한 요약 보고
```

사용: `/research 인증 처리 흐름`

## 스킬 설계 팁

### description은 명확하게

```
✅ description: 프로덕션 배포 체크리스트를 실행합니다.
❌ description: 배포
```

description이 명확할수록 자동 트리거 정확도가 높아집니다.

### 위험한 작업은 수동 전용으로

배포, 삭제 등은 반드시 `disable-model-invocation: true`를 설정하세요. AI가 임의로 실행하지 못하게 합니다.

### 동적 데이터 주입

`` !`command` `` 문법으로 스킬 실행 전 셸 명령어 결과를 주입할 수 있습니다:

```markdown
현재 브랜치: !`git branch --show-current`
변경된 파일: !`git diff --name-only`
```

## 관련 문서

- [.claude/hooks 활용](claude-hooks.md) — 자동 검증 (코드 수정 시 즉시 실행)
- [.claude/rules 작성](claude-rules.md) — 코딩 규칙 가이드라인
- [CLAUDE.md 작성](claude-md.md) — 프로젝트 맥락 정보

---

[← 목차로 돌아가기](../_sidebar.md)
