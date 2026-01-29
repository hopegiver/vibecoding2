# Claude Code - Git 작업 자동화

Git 커밋, PR 생성 등 버전 관리 자동화 방법을 학습합니다.

## Git 커밋

### 기본 커밋

```
변경 사항을 커밋해줘.

메시지: "Add user authentication"
```

**자동 수행:**
1. git status 확인
2. git diff 확인
3. 변경된 파일 add
4. 커밋 메시지 생성
5. git commit

### 여러 파일 커밋

```
src/auth/ 폴더의 모든 변경 사항을 커밋해줘.

메시지: "Implement JWT authentication"
```

## Pull Request 생성

### 기본 PR

```
PR을 만들어줘.

제목: "Add user authentication"
브랜치: feature/auth → main
```

**자동 수행:**
1. 현재 브랜치 확인
2. git log 확인
3. git diff main...HEAD 확인
4. PR 요약 작성
5. gh pr create 실행

### 상세 PR

```
PR을 만들어줘.

제목: "Add user authentication"
설명:
- JWT 토큰 기반 인증
- 로그인/로그아웃 API
- 미들웨어 구현
- 테스트 추가

리뷰어: @username
```

## 브랜치 관리

### 새 브랜치 생성

```
feature/new-api 브랜치를 만들어줘.
```

```bash
git checkout -b feature/new-api
```

### 브랜치 전환

```
main 브랜치로 전환해줘.
```

```bash
git checkout main
```

## 실전 프롬프트 예시

### 작업 완료 후 커밋

```
API 개발이 완료되었어.
변경 사항을 커밋하고 PR을 만들어줘.

커밋 메시지: "Add posts CRUD API"
PR 제목: "Implement posts API endpoints"
```

### Hotfix 배포

```
긴급 버그를 수정했어.

1. hotfix/critical-bug 브랜치 생성
2. 변경 사항 커밋
3. main으로 PR 생성
4. 리뷰어 추가: @team-lead
```

### 코드 리뷰 반영

```
코드 리뷰 피드백을 반영했어.

다음 변경 사항을 커밋:
- src/api.ts (타입 수정)
- src/utils.ts (함수명 변경)

메시지: "Fix: Apply code review feedback"
```

## Git 설정

### .claude/settings.local.json

```json
{
  "autoApproveTools": [
    "Bash(git status:*)",
    "Bash(git diff:*)",
    "Bash(git add:*)",
    "Bash(git commit:*)",
    "Bash(git push:*)"
  ]
}
```

**효과:** Git 명령 자동 승인

## 체크리스트

Git 작업 시 확인사항:

- [ ] 올바른 브랜치에 있는가?
- [ ] 커밋 메시지가 명확한가?
- [ ] 불필요한 파일은 제외했는가?
- [ ] PR 설명이 충분한가?
- [ ] 테스트가 통과하는가?

## 관련 문서

- [파일 읽기/쓰기 작업](file-operations.md)
- [터미널 명령 실행](terminal-commands.md)
