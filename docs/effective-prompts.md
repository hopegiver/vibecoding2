# 효과적인 프롬프트 작성 팁

## 개요

Claude Code에서 **더 나은 결과**를 얻기 위한 고급 프롬프트 작성 기법입니다.

## 핵심 원칙

### 1. 구체적으로 작성

**❌ 모호한 프롬프트:**
```
"API 만들어줘"
```

**✅ 구체적인 프롬프트:**
```
"User CRUD API를 만들어줘.
- Hono 프레임워크 사용
- D1 데이터베이스
- 5개 엔드포인트 (목록, 상세, 생성, 수정, 삭제)
- JWT 인증 필요
- KV 캐시 활용"
```

### 2. 컨텍스트 제공

**❌ 컨텍스트 없음:**
```
"ProductService 만들어줘"
```

**✅ 컨텍스트 포함:**
```
"현재 프로젝트는 Cloudflare Workers를 사용하고,
UserService와 동일한 패턴(클래스 기반, KV 캐시)을 따라.
src/services/userService.js를 참고해서 ProductService를 만들어줘."
```

### 3. 예제 제시

**❌ 예제 없음:**
```
"라우트 파일 만들어줘"
```

**✅ 예제 포함:**
```
"src/routes/users.js를 참고해서 src/routes/products.js를 만들어줘.
동일한 구조와 에러 처리 패턴을 사용해줘."
```

### 4. 제약사항 명시

**❌ 제약사항 없음:**
```
"로그인 API 만들어줘"
```

**✅ 제약사항 명시:**
```
"로그인 API를 만들어줘.

제약사항:
- bcrypt로 비밀번호 해싱 (Salt rounds: 10)
- JWT 토큰 (7일 만료)
- Rate Limiting (15분에 5번)
- 3회 실패 시 계정 잠금 (30분)
- 환경 변수에서 시크릿 읽기"
```

### 5. 단계별 진행

**❌ 한 번에 모든 것:**
```
"User, Product, Order CRUD를 모두 만들고,
인증, 권한, 결제, 알림 시스템까지 만들어줘"
```

**✅ 단계별 진행:**
```
"1단계: User CRUD API를 먼저 만들어줘.
(나중에 Product, Order는 순차적으로 요청할게)"
```

---

## 프롬프트 구조

### 기본 구조

```
[컨텍스트] + [작업] + [요구사항] + [제약사항]
```

**예제:**
```
[컨텍스트]
"현재 프로젝트는 Hono + D1 + KV를 사용하는 Workers 프로젝트야."

[작업]
"Product CRUD API를 만들어줘."

[요구사항]
"기능:
- GET /products - 목록 (페이징, 검색)
- GET /products/:id - 상세
- POST /products - 생성
- PUT /products/:id - 수정
- DELETE /products/:id - 삭제"

[제약사항]
"규칙:
- 서비스 레이어 패턴
- KV 캐시 우선
- 입력 검증 (Zod)"
```

### 고급 구조

```
[목표] + [컨텍스트] + [참고 자료] + [상세 요구사항] + [제약사항] + [체크리스트]
```

**예제:**
```
[목표]
"인증 시스템을 완전히 구축하고 싶어."

[컨텍스트]
"현재 프로젝트는 Cloudflare Workers 기반이고,
Hono 프레임워크를 사용하고 있어."

[참고 자료]
"기존 코드:
- src/middleware/auth.js (JWT 검증만 있음)
- src/routes/users.js (사용자 CRUD)"

[상세 요구사항]
"구현:
1. 회원가입 (POST /auth/register)
   - 이메일 검증
   - 비밀번호 해싱 (bcrypt)
   - 이메일 중복 체크

2. 로그인 (POST /auth/login)
   - 이메일/비밀번호 검증
   - JWT 토큰 발급 (7일)
   - Refresh 토큰 (30일)

3. 프로필 (GET /auth/profile)
   - JWT 검증 필요
   - 사용자 정보 반환

4. 로그아웃 (POST /auth/logout)
   - 토큰 블랙리스트 (KV)

5. 토큰 갱신 (POST /auth/refresh)
   - Refresh 토큰으로 새 Access 토큰 발급"

[제약사항]
"규칙:
- bcrypt Salt rounds: 10
- JWT는 환경 변수에서 시크릿 읽기
- Rate Limiting: 로그인은 15분에 5번
- 실패한 로그인 시도는 로깅
- 3회 실패 시 30분 계정 잠금
- 비밀번호는 최소 8자, 대소문자+숫자+특수문자"

[체크리스트]
"완료 후 확인:
- [ ] 보안 취약점 없음
- [ ] 에러 처리 완료
- [ ] 환경 변수 사용
- [ ] Rate Limiting 적용
- [ ] 로깅 추가"
```

---

## 고급 테크닉

### 1. 반복 개선

**첫 번째 프롬프트:**
```
"User 목록 API를 만들어줘"
```

**두 번째 프롬프트 (개선):**
```
"페이징 기능을 추가해줘. ?page=1&limit=20"
```

**세 번째 프롬프트 (추가 개선):**
```
"검색 기능도 추가해줘. ?search=keyword
name과 email에서 검색, 대소문자 구분 없이"
```

### 2. 조건부 요청

```
"src/services/productService.js를 확인하고,
- 파일이 있으면 getProductsByCategory 메소드만 추가
- 파일이 없으면 전체 ProductService 클래스 생성"
```

### 3. 비교 및 추천

```
"우리 프로젝트에서 세션 기반 인증과 JWT 중 어느 것이 더 나은지 비교해줘.

프로젝트 정보:
- Cloudflare Workers (stateless)
- 모바일 앱과 웹 모두 지원
- 예상 사용자: 10만명

비교 기준:
- 보안성
- 성능
- 확장성
- 구현 복잡도
- 비용

추천과 이유를 알려줘."
```

### 4. 다단계 작업

```
"다음 작업을 순서대로 진행해줘:

1. Product 테이블 D1 스키마 생성
   - migration 파일 생성
   - id, name, price, category, stock, created_at

2. ProductService 클래스 생성
   - CRUD 메소드
   - KV 캐시 활용
   - UserService 패턴 참고

3. Product 라우트 생성
   - 5개 엔드포인트
   - 인증 미들웨어 적용
   - admin만 생성/수정/삭제 가능

4. 단위 테스트 작성
   - ProductService 메소드별
   - Jest 사용

각 단계 완료 후 결과를 보여주고, 다음 단계 진행 전에 확인 요청해줘."
```

### 5. 검증 요청

```
"방금 생성한 코드를 검증해줘.

체크 항목:
- [ ] 보안: SQL Injection, XSS 취약점
- [ ] 성능: N+1 쿼리, 불필요한 순차 실행
- [ ] 에러 처리: 모든 예외 케이스 처리
- [ ] 코드 품질: 명명 규칙, 함수 길이, 중복 코드
- [ ] 테스트: 주요 기능 테스트 커버

문제가 있으면 수정해줘."
```

---

## 피해야 할 패턴

### 1. 너무 짧은 프롬프트

**❌:**
```
"API 만들어"
"버그 고쳐"
"테스트 작성"
```

**✅:**
```
"User CRUD API를 Hono + D1로 만들어줘"
"GET /products/:id에서 null 반환 버그를 고쳐줘"
"UserService.createUser()의 단위 테스트를 Jest로 작성해줘"
```

### 2. 모호한 표현

**❌:**
```
"좋은 코드로 만들어줘"
"성능을 개선해줘"
"보안을 강화해줘"
```

**✅:**
```
"DRY 원칙을 적용하고, 함수를 50줄 이내로 분리해줘"
"N+1 쿼리를 JOIN으로 변경하고, KV 캐시를 추가해줘"
"입력 검증, SQL Injection 방지, Rate Limiting을 추가해줘"
```

### 3. 컨텍스트 부족

**❌:**
```
"이 코드 리팩토링해줘"
(어떤 파일인지, 무엇을 개선할지 불명확)
```

**✅:**
```
"src/services/orderService.js의 createOrder 함수를 리팩토링해줘.
문제: 함수가 200줄로 너무 길고, 중복 코드가 많음.
개선: 작은 함수로 분리, DRY 원칙 적용"
```

### 4. 너무 많은 요구

**❌:**
```
"User, Product, Order, Payment, Notification, Analytics 시스템을
모두 만들고, 프론트엔드와 백엔드, 모바일 앱, 관리자 페이지까지 만들어줘"
```

**✅:**
```
"먼저 User CRUD API만 만들어줘.
나머지는 단계적으로 추가할게."
```

---

## 효과적인 질문 방법

### 1. 명확한 문제 제시

```
"이 에러를 해결해줘:

에러: TypeError: Cannot read property 'name' of undefined
위치: userService.js:45
발생 조건: getUser(999) 호출 시 (존재하지 않는 ID)

현재 코드:
[코드 붙여넣기]

원인을 찾아서 고쳐줘."
```

### 2. 비교 요청

```
"이 두 방법 중 어느 것이 더 나은지 비교해줘:

방법 1: forEach + async/await
users.forEach(async (user) => {
  await processUser(user);
});

방법 2: Promise.all
await Promise.all(users.map(user => processUser(user)));

비교 기준: 성능, 에러 처리, 가독성"
```

### 3. 설명 요청

```
"이 코드의 동작 원리를 설명해줘:

[코드 붙여넣기]

특히 다음을 중점적으로:
- KV 캐시가 어떻게 작동하는지
- 캐시 미스 시 어떻게 처리하는지
- TTL이 왜 3600초인지"
```

---

## 프롬프트 체크리스트

프롬프트 작성 전:

- [ ] **목표**: 무엇을 만들고 싶은가?
- [ ] **컨텍스트**: 프로젝트 정보를 제공했는가?
- [ ] **참고**: 기존 코드나 패턴을 언급했는가?
- [ ] **구체성**: 요구사항이 구체적인가?
- [ ] **제약사항**: 지켜야 할 규칙이 있는가?
- [ ] **검증**: 결과를 어떻게 확인할 것인가?

---

## 실전 예제

### Before: 비효과적인 프롬프트

```
"API 만들어줘"
```

### After: 효과적인 프롬프트

```
"Product CRUD API를 만들어줘.

컨텍스트:
- Cloudflare Workers 프로젝트
- Hono 프레임워크
- D1 데이터베이스
- UserService.js 패턴 참고

엔드포인트:
- GET /products?page=1&limit=20&search=&category= - 목록 (페이징, 검색, 필터)
- GET /products/:id - 상세
- POST /products - 생성 (admin만)
- PUT /products/:id - 수정 (admin만)
- DELETE /products/:id - 삭제 (admin만)

테이블 스키마:
- id INTEGER PRIMARY KEY
- name TEXT NOT NULL
- price INTEGER NOT NULL
- category TEXT NOT NULL
- stock INTEGER DEFAULT 0
- created_at TEXT NOT NULL

구현:
1. migrations/0002_create_products.sql (D1 마이그레이션)
2. src/services/productService.js (비즈니스 로직)
3. src/routes/products.js (API 엔드포인트)

규칙:
- 서비스 레이어 패턴 (클래스 기반)
- KV 캐시 우선 (TTL 1시간)
- 입력 검증 (price > 0, stock >= 0)
- admin 권한 체크 (생성/수정/삭제)
- 에러 처리 (NotFoundError, ValidationError)

완료 후:
- src/index.js에 라우트 등록
- 테스트 가능한 상태로"
```

---

## 다음 단계

- [자주 사용하는 프롬프트 패턴](prompt-patterns.md) - 기본 패턴
- [작업별 프롬프트 예시](common-scenarios.md) - 시나리오별 예제
- [컨텍스트 관리 및 체크포인트](context-management.md) - 장기 프로젝트

---

[← 목차로 돌아가기](../_sidebar.md)
