# Claude Code - 파일 읽기/쓰기 작업

Read, Write, Edit 도구를 활용한 파일 작업 방법을 학습합니다.

## Read 도구

### 기본 사용법

```
src/index.ts 파일을 읽어줘.
```

**결과:** 파일 전체 내용 표시

### 부분 읽기

```
src/index.ts 파일의 50~100줄을 읽어줘.
```

**활용:**
- 큰 파일의 특정 부분만 확인
- 토큰 절약

### 여러 파일 읽기

```
다음 파일들을 읽어줘:
- src/router.ts
- src/handlers/users.ts
- src/utils.ts
```

**팁:** 병렬로 읽기 때문에 빠름

## Write 도구

### 새 파일 생성

```
src/handlers/posts.ts 파일을 만들어줘.

내용:
- fetchPosts 함수
- createPost 함수
- updatePost 함수
- deletePost 함수

TypeScript 사용.
```

### 주의사항

- 기존 파일이 있으면 덮어씀
- 가능하면 Edit 사용 권장

## Edit 도구

### 특정 코드 수정

```
src/index.ts에서 다음을 수정해줘:

기존:
router.get('/api/users', ...)

변경:
router.get('/api/users', async (request, env) => {
    const users = await env.DB.prepare('SELECT * FROM users').all();
    return Response.json(users.results);
});
```

### 여러 곳 수정

```
src/index.ts에서 모든 console.log를 제거해줘.
```

**활용:**
- 변수명 변경
- Import 추가/제거
- 함수 시그니처 변경

## 실전 프롬프트 예시

### 컴포넌트 생성

```
src/components/Header.tsx 파일을 만들어줘.

React 컴포넌트:
- Props: title (string)
- 네비게이션 메뉴 (Home, About, Contact)
- 반응형 디자인
- TypeScript

Tailwind CSS 사용.
```

### 설정 파일 수정

```
package.json을 읽고 다음 스크립트를 추가해줘:

"scripts": {
    "test": "vitest",
    "lint": "eslint .",
    "format": "prettier --write ."
}
```

### 리팩토링

```
src/utils.ts를 읽어줘.

함수들을 다음 파일로 분리:
- src/utils/string.ts (문자열 관련)
- src/utils/date.ts (날짜 관련)
- src/utils/validation.ts (유효성 검사)
```

## 체크리스트

파일 작업 시 확인사항:

- [ ] 파일 경로가 정확한가?
- [ ] 기존 파일을 덮어쓰지 않는가?
- [ ] 필요한 import가 추가되었는가?
- [ ] 코드 스타일이 일관적인가?
- [ ] 수정 후 동작 확인을 했는가?

## 관련 문서

- [코드 검색 및 탐색](code-navigation.md)
- [Git 작업 자동화](git-automation.md)
