# Claude Code - 터미널 명령 실행

Bash 도구를 활용한 터미널 작업 자동화 방법을 학습합니다.

## Bash 도구 기본

### 단순 명령

```
현재 디렉토리의 파일 목록을 보여줘.
```

```bash
ls -la
```

### 복합 명령

```
프로젝트를 빌드하고 테스트를 실행해줘.
```

```bash
npm run build && npm test
```

## npm/yarn 작업

### 패키지 설치

```
다음 패키지를 설치해줘:
- typescript
- @types/node
- vitest
```

```bash
npm install -D typescript @types/node vitest
```

### 스크립트 실행

```
개발 서버를 시작해줘.
```

```bash
npm run dev
```

## 빌드 및 배포

### 빌드

```
프로젝트를 프로덕션 모드로 빌드해줘.
```

```bash
npm run build
```

### 배포

```
Cloudflare Workers에 배포해줘.
```

```bash
npx wrangler deploy
```

## 데이터베이스 작업

### 마이그레이션

```
D1 스키마를 적용해줘.
```

```bash
npx wrangler d1 execute mydb --file=schema.sql
```

### 데이터 조회

```
D1 데이터베이스에서 사용자 목록을 조회해줘.
```

```bash
npx wrangler d1 execute mydb --command="SELECT * FROM users"
```

## 실전 프롬프트 예시

### 프로젝트 초기화

```
새 프로젝트를 초기화해줘.

단계:
1. npm init -y
2. TypeScript 설치
3. ESLint, Prettier 설정
4. Git 초기화
5. .gitignore 생성
```

### 테스트 자동화

```
다음을 순서대로 실행해줘:

1. 린트 체크 (npm run lint)
2. 타입 체크 (tsc --noEmit)
3. 테스트 실행 (npm test)
4. 빌드 (npm run build)

오류가 있으면 중단하고 알려줘.
```

### 로그 분석

```
Wrangler tail로 실시간 로그를 확인해줘.

필터:
- 에러만 표시
- POST 요청만 표시
```

```bash
npx wrangler tail --status error --method POST
```

## 백그라운드 실행

### 개발 서버

```
개발 서버를 백그라운드로 실행해줘.
```

```bash
npm run dev &
```

**확인:**
```
ps aux | grep "npm run dev"
```

## 체크리스트

터미널 작업 시 확인사항:

- [ ] 올바른 디렉토리에 있는가?
- [ ] 필요한 권한이 있는가?
- [ ] 명령어가 정확한가?
- [ ] 오류 처리가 되어 있는가?
- [ ] 로그를 확인했는가?

## 관련 문서

- [파일 읽기/쓰기 작업](file-operations.md)
- [Git 작업 자동화](git-automation.md)
