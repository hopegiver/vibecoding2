# .claude/commands 활용 가이드

## 개요

`.claude/commands/`는 **사용자 정의 슬래시 커맨드**를 저장하는 폴더입니다. 반복적인 작업을 간단한 명령어로 자동화하여 생산성을 극대화합니다.

## 왜 필요한가?

### Commands 없이 작업할 때

```
개발자: "Product CRUD API를 만들어줘.
- REST 엔드포인트 (GET, POST, PUT, DELETE)
- D1 데이터베이스 연동
- JWT 인증
- 페이징 10개씩
- 검색 기능
- 정렬 기능
- 에러 핸들링
- 입력 검증
- 로깅
- 테스트 코드
- API 문서"

(매번 긴 프롬프트 작성...)
```

### Commands 사용 시

```
개발자: "/crud Product"

Claude: "Product CRUD API를 생성합니다.
- src/routes/products.js ✅
- src/services/productService.js ✅
- src/tests/product.test.js ✅
- docs/api/products.md ✅

모든 기능 완료!"
```

## 기본 구조

```
.claude/commands/
├── crud.md          # CRUD API 생성
├── page.md          # 페이지 생성
├── dao.md           # DAO 클래스 생성
├── review.md        # 코드 리뷰
├── deploy.md        # 배포 체크리스트
└── refactor.md      # 리팩토링
```

## 슬래시 커맨드 작성법

### 기본 템플릿

**파일: `.claude/commands/example.md`**

```markdown
# 커맨드 제목

커맨드 설명 (한 줄 요약)

## 수행 작업

1. 첫 번째 작업
2. 두 번째 작업
3. 세 번째 작업

## 규칙

- 규칙 1
- 규칙 2

## 체크리스트

- [ ] 체크 항목 1
- [ ] 체크 항목 2
```

### MCP 자동 참조 패턴

**`.claude/commands/crud.md`**

```markdown
# CRUD API 생성

@malgn-docs와 @company-wiki를 참고하여 표준 CRUD API를 생성합니다.

## 수행 작업

1. **문서 확인**
   - @malgn-docs에서 API 패턴 확인
   - @company-wiki에서 보안 규칙 확인

2. **파일 생성**
   - src/routes/{리소스명}.js
   - src/services/{리소스명}Service.js
   - src/tests/{리소스명}.test.js

3. **기능 구현**
   - GET /{리소스명} - 목록 (페이징 10개)
   - GET /{리소스명}/:id - 상세
   - POST /{리소스명} - 생성
   - PUT /{리소스명}/:id - 수정
   - DELETE /{리소스명}/:id - 삭제

4. **보안 및 검증**
   - JWT 인증 적용
   - 입력 검증
   - XSS 방지
   - SQL Injection 방지

5. **테스트 및 문서**
   - 유닛 테스트 작성
   - API 문서 생성

## 규칙

- 에러 응답 형식: `{ error: "메시지", code: "ERROR_CODE" }`
- 성공 응답 형식: `{ data: {...}, meta: {...} }`
- 모든 엔드포인트에 로깅 추가
- 페이징 기본값: 10개

## 체크리스트

- [ ] 모든 엔드포인트 구현
- [ ] 인증 적용
- [ ] 테스트 통과
- [ ] API 문서 작성
- [ ] 에러 핸들링 완료
```

**사용:**
```
/crud Product
```

## 실전 예제

### 예제 1: 맑은프레임워크 페이지 생성

**`.claude/commands/malgn-page.md`**

```markdown
# 맑은프레임워크 페이지 생성

@malgn-docs를 참고하여 표준 JSP 페이지를 생성합니다.

## 수행 작업

1. **DAO 생성**
   - src/dao/{이름}Dao.java
   - CRUD 메소드 구현

2. **JSP 생성**
   - public_html/{폴더}/{이름}_list.jsp - 목록
   - public_html/{폴더}/{이름}_form.jsp - 등록/수정
   - public_html/{폴더}/{이름}_view.jsp - 상세

3. **HTML 템플릿 생성**
   - html/{폴더}/list.html
   - html/{폴더}/form.html
   - html/{폴더}/view.html

## 패턴

**JSP 구조:**
```jsp
<%@ page import="..." %>
<%
Malgn m = new Malgn(request, response, out);
Form f = new Form();
Page p = new Page();
Auth auth = new Auth(request, response);

// POST 처리
if(m.isPost() && f.validate()) {
    // 처리 로직
    return;
}

// GET 처리
p.setLayout("default");
p.setBody("{경로}");
p.display();
%>
```

## 규칙

- JSP에 HTML 직접 작성 금지
- try-catch 사용 금지
- POST 후 반드시 return
- DataSet 사용 전 next() 호출
- XSS 필터링: m.rs() 사용

## 체크리스트

- [ ] DAO 클래스 작성
- [ ] JSP 3개 파일 생성
- [ ] HTML 템플릿 3개 생성
- [ ] Postback 패턴 적용
- [ ] 보안 규칙 준수
```

**사용:**
```
/malgn-page Product board
```

### 예제 2: 코드 리뷰

**`.claude/commands/review.md`**

```markdown
# 코드 리뷰

@malgn-docs와 @company-wiki를 기준으로 코드를 리뷰합니다.

## 수행 작업

1. **변경 사항 확인**
   - git diff로 수정된 파일 확인
   - 주요 변경 내용 파악

2. **코딩 표준 검증**
   - @malgn-docs의 패턴 준수 여부
   - @company-wiki의 규칙 준수 여부

3. **보안 검토**
   - XSS, SQL Injection 취약점
   - 인증/권한 체크 누락
   - 민감 정보 노출

4. **성능 검토**
   - N+1 쿼리 문제
   - 불필요한 DB 호출
   - 캐싱 누락

5. **테스트 검토**
   - 테스트 커버리지
   - Edge case 처리

## 리뷰 형식

**파일별 리뷰:**
```
📄 src/services/productService.js

✅ 좋은 점:
- 명확한 함수명
- 에러 핸들링 완벽

⚠️ 개선 필요:
- Line 45: N+1 쿼리 문제
  → Promise.all로 병렬 처리 권장
- Line 78: 하드코딩된 상수
  → config 파일로 이동

🔴 Critical:
- Line 92: SQL Injection 위험
  → PreparedStatement 사용 필수
```

## 체크리스트

- [ ] 모든 파일 리뷰 완료
- [ ] Critical 이슈 0개
- [ ] 보안 취약점 해결
- [ ] 성능 이슈 확인
- [ ] 테스트 커버리지 80% 이상
```

**사용:**
```
/review
```

### 예제 3: 배포 체크리스트

**`.claude/commands/deploy.md`**

```markdown
# 배포 체크리스트

프로덕션 배포 전 필수 체크 항목을 확인합니다.

## 수행 작업

1. **린트 검사**
   ```bash
   npm run lint
   ```

2. **타입 체크**
   ```bash
   tsc --noEmit
   ```

3. **테스트 실행**
   ```bash
   npm test
   ```

4. **빌드**
   ```bash
   npm run build
   ```

5. **보안 스캔**
   ```bash
   npm audit
   ```

6. **환경 변수 확인**
   - .env.production 확인
   - 필수 변수 누락 여부

7. **배포**
   ```bash
   npx wrangler deploy
   ```

8. **배포 후 확인**
   - 헬스체크 API 호출
   - 주요 기능 smoke test
   - 로그 확인

## 자동 중단 조건

다음 중 하나라도 실패하면 배포 중단:
- 린트 에러
- 타입 에러
- 테스트 실패
- 빌드 실패
- Critical 보안 취약점

## 체크리스트

- [ ] 모든 테스트 통과
- [ ] 빌드 성공
- [ ] 보안 스캔 Clean
- [ ] 환경 변수 설정 완료
- [ ] 배포 성공
- [ ] Smoke test 통과
```

**사용:**
```
/deploy
```

### 예제 4: 리팩토링

**`.claude/commands/refactor.md`**

```markdown
# 리팩토링

@malgn-docs와 @company-wiki의 최신 패턴으로 코드를 개선합니다.

## 수행 작업

1. **현재 코드 분석**
   - 코드 스멜 찾기
   - 중복 코드 식별
   - 복잡도 측정

2. **리팩토링 계획**
   - 개선 항목 우선순위 정렬
   - 위험도 평가

3. **리팩토링 실행**
   - 테스트 먼저 작성 (없는 경우)
   - 단계별 리팩토링
   - 각 단계마다 테스트 확인

4. **문서 업데이트**
   - 주요 변경 사항 기록
   - .claude/memory 업데이트

## 리팩토링 패턴

### 중복 코드 제거
```javascript
// Before
function getUserById(id) { /* 반복되는 로직 */ }
function getProductById(id) { /* 반복되는 로직 */ }

// After
function getEntityById(entity, id) { /* 공통 로직 */ }
```

### 복잡한 조건문 개선
```javascript
// Before
if (user && user.role === 'admin' && user.active && !user.banned) {
  // ...
}

// After
function isAuthorizedAdmin(user) {
  return user?.role === 'admin' && user?.active && !user?.banned;
}
if (isAuthorizedAdmin(user)) {
  // ...
}
```

## 규칙

- 한 번에 하나씩 리팩토링
- 각 단계마다 테스트 실행
- 동작 변경 금지 (behavior preserving)
- 커밋을 작게 자주

## 체크리스트

- [ ] 리팩토링 계획 수립
- [ ] 테스트 작성/확인
- [ ] 단계별 리팩토링 완료
- [ ] 모든 테스트 통과
- [ ] 문서 업데이트
- [ ] 코드 리뷰
```

**사용:**
```
/refactor src/services/userService.js
```

## 고급 패턴

### 패턴 1: 파라미터 받기

**`.claude/commands/api.md`**

```markdown
# API 엔드포인트 생성

단일 API 엔드포인트를 생성합니다.

사용법: /api {리소스명} {메소드} {경로}

예: /api Product GET /products/:id

## 수행 작업

1. 파라미터 파싱
   - 리소스명: {리소스명}
   - HTTP 메소드: {메소드}
   - 경로: {경로}

2. 파일 생성
   - src/routes/{리소스명}.js에 추가

3. 기능 구현
   - 라우트 핸들러
   - 서비스 호출
   - 응답 포맷팅
```

### 패턴 2: 조건부 실행

**`.claude/commands/test.md`**

```markdown
# 테스트 실행

조건에 따라 다른 테스트를 실행합니다.

## 수행 작업

1. **변경된 파일 확인**
   ```bash
   git diff --name-only
   ```

2. **조건부 테스트**
   - src/services/ 변경 → 서비스 테스트만
   - src/routes/ 변경 → API 테스트만
   - 전체 변경 → 전체 테스트

3. **테스트 실행**
   ```bash
   npm test -- {필터}
   ```

4. **결과 리포트**
   - 통과/실패 개수
   - 커버리지
   - 실행 시간
```

### 패턴 3: 다른 Commands 연계

**`.claude/commands/full-feature.md`**

```markdown
# 완전한 기능 구현

CRUD + 테스트 + 문서 + 배포까지 한 번에 수행합니다.

## 수행 작업

1. **CRUD 생성**
   - /crud {리소스명} 실행

2. **코드 리뷰**
   - /review 실행

3. **테스트**
   - /test 실행

4. **문서 생성**
   - API 문서 자동 생성

5. **Memory 업데이트**
   - /save-progress 실행

6. **배포 준비**
   - /deploy 실행 (dry-run)
```

## Commands 관리 팁

### 1. 명확한 네이밍

```
✅ 좋은 예:
- create-api.md
- review-security.md
- deploy-production.md

❌ 나쁜 예:
- api.md (너무 일반적)
- do-stuff.md (목적 불명확)
```

### 2. 문서화

각 Command 상단에 명확한 설명과 사용 예시 추가:

```markdown
# CRUD API 생성

표준 RESTful CRUD API를 자동 생성합니다.

**사용법:**
```
/crud {리소스명}
```

**예시:**
```
/crud Product
/crud User
```
```

### 3. 버전 관리

프로젝트마다 다른 Commands 사용:

```
프로젝트A/.claude/commands/
├── crud-rest.md       # REST API 패턴
└── page-react.md      # React 페이지

프로젝트B/.claude/commands/
├── crud-graphql.md    # GraphQL 패턴
└── page-vue.md        # Vue 페이지
```

### 4. 팀 공유

Commands를 Git에 커밋하여 팀 전체가 활용:

```bash
git add .claude/commands/
git commit -m "Add: 표준 개발 Commands"
git push
```

## 자주 사용하는 Commands 모음

### 개발 Commands

1. `/crud` - CRUD API 생성
2. `/page` - 페이지 생성
3. `/dao` - DAO 클래스 생성
4. `/component` - 컴포넌트 생성
5. `/api` - 단일 API 엔드포인트

### 품질 Commands

1. `/review` - 코드 리뷰
2. `/refactor` - 리팩토링
3. `/test` - 테스트 실행
4. `/lint` - 린트 검사
5. `/security` - 보안 스캔

### 관리 Commands

1. `/deploy` - 배포 체크리스트
2. `/save-progress` - 진행 상황 저장
3. `/sync` - 팀 동기화
4. `/docs` - 문서 생성
5. `/cleanup` - 코드 정리

## Commands 활용 체크리스트

- [ ] 자주 하는 작업을 Command로 만들었는가?
- [ ] 각 Command에 명확한 설명이 있는가?
- [ ] MCP 서버를 활용하는가?
- [ ] 팀과 Commands를 공유했는가?
- [ ] Commands를 정기적으로 업데이트하는가?

## 관련 문서

- [.claude/memory 활용](claude-memory.md) - 작업 이력 관리
- [MCP 서버 설정](mcp-setup.md) - Commands에서 MCP 활용
- [.claude/rules 작성](claude-rules.md) - 코딩 규칙 정의

---

[← 목차로 돌아가기](../_sidebar.md)
