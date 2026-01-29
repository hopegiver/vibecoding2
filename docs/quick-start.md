# 5분 안에 시작하기

## Claude Code란?

Claude Code는 Anthropic의 AI 모델 Claude를 기반으로 한 **AI 페어 프로그래머**입니다. VSCode에서 자연어로 대화하며 코드를 작성, 수정, 리팩토링할 수 있습니다.

## 바이브코딩이란?

**바이브코딩(Vibe Coding)**은 AI와 협업하여 개발하는 새로운 방식입니다:

- 💬 자연어로 코드 요청
- 🤖 AI가 프로젝트 컨텍스트를 이해하고 일관된 코드 생성
- ⚡ 반복 작업 자동화
- 🎯 개발자는 핵심 로직에 집중

## 1분: Claude Code 설치

### VSCode 확장 설치

1. VSCode 열기
2. 확장(Extensions) 탭 열기 (`Ctrl+Shift+X`)
3. "Claude Code" 검색
4. **Install** 클릭
5. Anthropic API 키 입력 (가입 필요)

## 2분: 첫 프롬프트 실행

### 예제 1: 파일 읽기

```
프롬프트: "src/index.js 파일 읽어줘"
```

**Claude Code 동작:**
- 파일을 읽고 내용 분석
- 코드 구조 설명
- 개선 제안

### 예제 2: 코드 생성

```
프롬프트: "users 테이블에 대한 CRUD API를 만들어줘.
Hono 프레임워크를 사용하고, D1 데이터베이스를 연결해줘."
```

**Claude Code 동작:**
- `src/routes/users.js` 생성 (라우트)
- `src/services/userService.js` 생성 (비즈니스 로직)
- `src/index.js` 업데이트 (라우트 등록)

### 예제 3: 리팩토링

```
프롬프트: "이 함수를 async/await 패턴으로 리팩토링해줘"
```

**Claude Code 동작:**
- Promise then/catch → async/await 변환
- 에러 처리 개선
- 코드 가독성 향상

### 예제 4: 버그 수정

```
프롬프트: "이 코드에서 왜 null이 반환되는지 찾아서 고쳐줘"
```

**Claude Code 동작:**
- 코드 분석
- 문제점 발견 및 설명
- 수정 코드 제안

### 예제 5: 테스트 작성

```
프롬프트: "UserService의 getUser 메소드에 대한 테스트를 작성해줘"
```

**Claude Code 동작:**
- 테스트 파일 생성
- 다양한 케이스 커버 (성공, 실패, 엣지 케이스)
- Mock 데이터 포함

## 2분: 프로젝트 컨텍스트 설정

Claude Code가 더 정확한 코드를 생성하려면 **프로젝트 컨텍스트**가 필요합니다.

### 최소 설정: CLAUDE.md 생성

프로젝트 루트에 `CLAUDE.md` 파일 생성:

```markdown
# 프로젝트명

## 기술 스택
- Node.js + Hono
- Cloudflare Workers
- D1 Database

## 프로젝트 구조
\`\`\`
src/
├── routes/         # API 라우트
├── services/       # 비즈니스 로직
└── index.js        # 엔트리 포인트
\`\`\`

## 핵심 규칙
- 서비스는 클래스 기반으로 작성
- 라우트에는 비즈니스 로직 금지
- 파일명은 camelCase 사용
```

### 프롬프트 재실행

```
프롬프트: "CLAUDE.md를 읽고, users 테이블 CRUD API를 만들어줘"
```

이제 Claude Code가 **프로젝트 규칙을 따르는 코드**를 생성합니다!

## 바이브코딩 핵심 팁

### 1. 명확하게 요청하기

❌ **나쁜 예:**
```
"로그인 만들어줘"
```

✅ **좋은 예:**
```
"JWT 기반 로그인 API를 만들어줘.
- POST /auth/login 엔드포인트
- email, password 검증
- JWT 토큰 발급
- 토큰 유효기간: 7일"
```

### 2. 컨텍스트 제공하기

```
프롬프트: "현재 프로젝트는 Hono 프레임워크를 사용하고 있어.
UserService 클래스를 참고해서 ProductService를 만들어줘."
```

### 3. 예제 제시하기

```
프롬프트: "기존 users.js 라우트 파일을 참고해서
products.js 라우트를 만들어줘."
```

### 4. 점진적으로 개선하기

```
프롬프트 1: "사용자 목록 조회 API 만들어줘"
프롬프트 2: "페이징 기능 추가해줘"
프롬프트 3: "검색 기능도 추가해줘"
```

## 다음 단계

- [개발 환경 설정](setup.md) - 프로젝트 초기 설정 상세 가이드
- [첫 프로젝트 만들기](first-project.md) - 실전 CRUD 프로젝트 생성
- [.claude/rules 작성 가이드](claude-rules.md) - 프로젝트 규칙 정의
- [CLAUDE.md 작성 가이드](claude-md.md) - 프로젝트 컨텍스트 문서

---

[← 목차로 돌아가기](../_sidebar.md)
