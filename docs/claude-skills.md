# .claude/skills 활용 가이드

## 개요

`.claude/skills/`는 **사용자 정의 스킬**을 저장하는 폴더입니다. 반복적인 작업을 간단한 명령어로 자동화하고, Claude가 상황에 따라 **자동으로 스킬을 호출**하게 만들 수 있습니다.

> **참고:** 기존 `.claude/commands/` 방식도 계속 작동합니다. Skills는 Commands의 상위 호환으로, 추가 기능을 제공합니다.

## Commands vs Skills

| 항목 | `.claude/commands/*.md` | `.claude/skills/name/SKILL.md` |
|------|------------------------|-------------------------------|
| 슬래시 명령어 | ✅ `/명령어` | ✅ `/명령어` |
| 자동 트리거 | ❌ | ✅ description 기반 |
| 지원 파일 (예제, 스크립트 등) | ❌ | ✅ 폴더 내 자유롭게 추가 |
| 모델/노력 수준 제어 | ❌ | ✅ `model`, `effort` 필드 |
| Subagent 실행 | ❌ | ✅ `context: fork` |
| 동적 컨텍스트 주입 | ❌ | ✅ `` !`command` `` 전처리 |
| 경로 기반 활성화 | ❌ | ✅ `paths` 필드 |

## 기본 구조

```
프로젝트루트/
└── .claude/
    └── skills/
        ├── crud/
        │   └── SKILL.md
        ├── review/
        │   └── SKILL.md
        └── deploy/
            ├── SKILL.md
            └── checklist.md    # 지원 파일 자유롭게 추가 가능
```

> **범위:** `~/.claude/skills/`에 두면 모든 프로젝트에서 사용, `.claude/skills/`에 두면 현재 프로젝트만.

## SKILL.md 작성법

### 기본 형식

```markdown
---
name: skill-name
description: 이 스킬이 무엇을 하는지 설명. Claude가 자동 호출 여부를 이 설명으로 판단합니다.
---

스킬 지시사항을 여기에 작성합니다.

$ARGUMENTS 위치에 사용자가 전달한 인자가 들어옵니다.
```

### Frontmatter 필드

| 필드 | 설명 | 기본값 |
|------|------|-------|
| `name` | 슬래시 명령어 이름 (생략 시 폴더명) | 폴더명 |
| `description` | 자동 트리거 판단 기준. 250자 이내 권장 | 첫 단락 |
| `disable-model-invocation` | `true`면 사용자만 실행 가능 (자동 호출 방지) | `false` |
| `user-invocable` | `false`면 Claude만 호출 가능 (사용자에게 숨김) | `true` |
| `allowed-tools` | 이 스킬 실행 시 허용할 도구 | 세션 설정 |
| `model` | 실행 시 사용할 모델 | 세션 모델 |
| `effort` | 노력 수준 (`low` / `medium` / `high` / `max`) | 세션 설정 |
| `context` | `fork`면 새 subagent에서 독립 실행 | 인라인 |
| `agent` | `context: fork` 시 사용할 agent 타입 | `general-purpose` |
| `paths` | 특정 파일 패턴에서만 활성화 (예: `"*.py"`) | 전체 |

## 좋은 스킬 설계 원칙

스킬은 단순히 "무엇을 해라"는 지시가 아니라, **작업 절차와 완료 기준**을 함께 정의해야 합니다.

### 1. 사전 가이드 포함

스킬 본문에 **단계별 절차**를 명시하면 Claude가 작업 순서를 지키며 일관된 결과를 냅니다.

```markdown
## 작업 절차

1. 기존 코드 파악 (Read로 관련 파일 읽기)
2. 작업 계획 수립 및 사용자 확인
3. 구현 (변경 최소화 원칙 준수)
4. 완료 체크리스트 검증
```

> 절차가 없으면 Claude가 매번 다른 방식으로 접근합니다. 절차를 문서화하면 팀 전체가 예측 가능한 결과를 얻습니다.

### 2. 완료 후 체크리스트 포함

작업이 끝나면 Claude가 **스스로 체크리스트를 검증**하고 결과를 보고하도록 지시합니다.

```markdown
## 완료 체크리스트

작업 완료 후 아래 항목을 반드시 확인하고 결과를 보고하세요:

- [ ] 요청한 파일이 모두 생성/수정되었는가?
- [ ] 기존 코드와 일관된 스타일인가?
- [ ] 에러 처리가 누락되지 않았는가?
- [ ] 불필요한 변경이 없는가?
```

> 체크리스트를 스킬에 넣으면 Claude가 작업 후 자동으로 검증합니다. 사용자가 매번 확인하지 않아도 됩니다.

### 3. 구조 템플릿

```markdown
---
name: skill-name
description: 스킬 설명
---

## 개요

이 스킬이 하는 일과 언제 사용하는지 간략히 설명합니다.

## 작업 절차

1. 첫 번째 단계
2. 두 번째 단계
3. 세 번째 단계

## 구현 규칙

- 지켜야 할 규칙 1
- 지켜야 할 규칙 2

## 완료 체크리스트

작업 완료 후 아래 항목을 확인하고 결과를 보고하세요:

- [ ] 체크 항목 1
- [ ] 체크 항목 2
- [ ] 체크 항목 3
```

---

## 실전 예시

### 예시 1: CRUD API 생성

**`.claude/skills/crud/SKILL.md`**

```markdown
---
name: crud
description: 표준 RESTful CRUD API를 생성합니다. REST API, 엔드포인트, 서비스 레이어 생성 요청 시 사용합니다.
disable-model-invocation: true
allowed-tools: Read Write Edit Bash
---

# CRUD API 생성

$ARGUMENTS 리소스에 대한 표준 CRUD API를 생성합니다.

## 생성 파일

- `src/routes/$ARGUMENTS.js` - 라우트 핸들러
- `src/services/${ARGUMENTS}Service.js` - 비즈니스 로직
- `src/tests/$ARGUMENTS.test.js` - 테스트 코드

## 구현 규칙

- GET /$ARGUMENTS - 목록 (페이징 10개)
- GET /$ARGUMENTS/:id - 상세
- POST /$ARGUMENTS - 생성
- PUT /$ARGUMENTS/:id - 수정
- DELETE /$ARGUMENTS/:id - 삭제
- 에러 응답: `{ error: "메시지", code: "ERROR_CODE" }`
- 성공 응답: `{ data: {...}, meta: {...} }`
- JWT 인증 적용
- 입력 검증 및 XSS 방지
```

**사용:**
```
/crud Product
/crud User
```

---

### 예시 2: 코드 리뷰 (자동 트리거)

**`.claude/skills/review/SKILL.md`**

```markdown
---
name: review
description: 코드 품질, 보안, 성능을 리뷰합니다. PR 리뷰, 코드 검토, 품질 분석 요청 시 자동으로 사용합니다.
allowed-tools: Read Grep Glob
---

## 코드 리뷰 체크리스트

다음 항목을 순서대로 검토합니다:

1. **에러 처리** - 예외 처리가 적절한가?
2. **보안** - SQL Injection, XSS, 인증 누락 없는가?
3. **성능** - N+1 쿼리, 불필요한 루프 없는가?
4. **가독성** - 함수명, 변수명이 명확한가?
5. **테스트** - 테스트 커버리지가 충분한가?

파일 경로와 줄 번호를 명시하여 구체적으로 피드백합니다.
```

> `disable-model-invocation`을 설정하지 않았으므로, 사용자가 "코드 리뷰해줘"라고 말하면 Claude가 자동으로 이 스킬을 호출합니다.

---

### 예시 3: 배포 (수동 전용 + 동적 컨텍스트)

**`.claude/skills/deploy/SKILL.md`**

```markdown
---
name: deploy
description: 프로덕션 배포 체크리스트를 실행합니다.
disable-model-invocation: true
allowed-tools: Bash Read
effort: high
---

# 배포 체크리스트

현재 git 상태: !`git status --short`
최근 커밋: !`git log --oneline -5`

## 배포 대상
$ARGUMENTS

## 단계

1. 테스트 통과 확인
2. 빌드 실행
3. 환경 변수 점검
4. 배포 실행
5. 헬스체크 확인
```

**사용:**
```
/deploy production
/deploy staging
```

> `` !`command` `` 문법으로 스킬 실행 전 셸 명령어를 실행하여 동적 데이터를 주입합니다.

---

### 예시 4: 독립 Subagent 실행

**`.claude/skills/research/SKILL.md`**

```markdown
---
name: research
description: 코드베이스의 특정 주제를 독립적으로 깊이 조사합니다.
context: fork
agent: Explore
---

## 조사 주제

$ARGUMENTS에 대해 코드베이스 전체를 분석합니다.

1. 관련 파일 탐색
2. 코드 패턴 분석
3. 파일 경로와 줄 번호를 포함한 요약 보고

메인 대화 컨텍스트에 영향을 주지 않고 독립 실행합니다.
```

**사용:**
```
/research 인증 처리 흐름
```

> `context: fork`로 설정하면 별도 subagent에서 실행되므로, 메인 컨텍스트가 오염되지 않습니다.

---

### 예시 5: 특정 파일에서만 활성화

**`.claude/skills/django-model/SKILL.md`**

```markdown
---
name: django-model
description: Django 모델 생성 및 마이그레이션을 지원합니다.
paths: "*.py,**/models/**"
allowed-tools: Read Write Edit Bash
---

## Django 모델 생성

$ARGUMENTS 모델을 Django 표준으로 생성합니다.

- `models.py`에 모델 클래스 추가
- `admin.py`에 관리자 등록
- 마이그레이션 파일 생성: `python manage.py makemigrations`
```

> Python 파일 또는 models 폴더에서 작업할 때만 이 스킬이 활성화됩니다.

## 자동 트리거 원리

Claude는 세션 시작 시 모든 스킬의 `description`을 로드하고, 사용자의 요청과 의미가 매칭되면 자동으로 해당 스킬을 적용합니다.

```
세션 시작
  → 모든 SKILL.md의 description 로드
  → 사용자 메시지 분석
  → description과 의미 매칭
  → 매칭 시 전체 SKILL.md 로드 및 적용
```

**자동 트리거 제어:**

```markdown
---
# 항상 자동 호출 허용 (기본값)
disable-model-invocation: false

# 사용자가 /명령어로만 실행 가능
disable-model-invocation: true

# Claude만 호출 가능, 사용자에게 숨김
user-invocable: false
---
```

## 마이그레이션: Commands → Skills

기존 `.claude/commands/crud.md`가 있다면:

```bash
# 디렉토리 생성
mkdir -p .claude/skills/crud

# 파일 이동 (내용 동일, 경로만 변경)
mv .claude/commands/crud.md .claude/skills/crud/SKILL.md
```

내용은 그대로 사용할 수 있으며, frontmatter에 필드를 추가하면 Skills 기능을 활용할 수 있습니다.

## Skills 관리 팁

### 명확한 description 작성

```markdown
✅ 좋은 예:
description: CRUD API를 생성합니다. REST 엔드포인트, 서비스 레이어 생성 요청 시 사용합니다.

❌ 나쁜 예:
description: API 생성
```

description이 명확할수록 자동 트리거 정확도가 높아집니다.

### 수동 실행만 원할 때

배포, 삭제 등 위험한 작업은 반드시 `disable-model-invocation: true` 설정:

```markdown
---
name: drop-table
description: 데이터베이스 테이블을 삭제합니다.
disable-model-invocation: true   # 사용자가 직접 /drop-table로만 실행
---
```

### 팀 공유

Skills를 Git에 커밋하면 팀 전체가 동일한 스킬을 사용할 수 있습니다:

```bash
git add .claude/skills/
git commit -m "Add: 표준 개발 Skills"
git push
```

## 자주 사용하는 Skills 예시

### 개발
- `/crud {리소스}` - CRUD API 생성
- `/page {이름}` - 페이지 생성
- `/component {이름}` - 컴포넌트 생성

### 품질
- `/review` - 코드 리뷰 (또는 자동 트리거)
- `/refactor` - 리팩토링
- `/security` - 보안 스캔

### 관리
- `/deploy {환경}` - 배포 체크리스트
- `/research {주제}` - 코드베이스 조사

## Skills 활용 체크리스트

- [ ] 자주 반복하는 작업을 Skill로 만들었는가?
- [ ] description이 명확하여 자동 트리거가 올바르게 작동하는가?
- [ ] 위험한 작업에 `disable-model-invocation: true`를 설정했는가?
- [ ] 팀과 Skills를 Git으로 공유했는가?
- [ ] 불필요한 자동 트리거가 발생하지 않는가?

## 관련 문서

- [.claude/memory 활용](claude-memory.md) - 작업 이력 관리
- [MCP 서버 설정](mcp-setup.md) - Skills에서 MCP 활용
- [.claude/rules 작성](claude-rules.md) - 코딩 규칙 정의

---

[← 목차로 돌아가기](../_sidebar.md)
