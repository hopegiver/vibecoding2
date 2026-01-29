# 작업별 프롬프트 예시

## 개요

실무에서 자주 발생하는 **시나리오별 프롬프트 예제**를 제공합니다.

## 프로젝트 초기 설정

### 시나리오 1: 새 프로젝트 시작

```
프롬프트: "Cloudflare Workers 프로젝트를 초기 설정해줘.

구조:
- Hono 프레임워크
- D1 데이터베이스
- KV 캐시
- 서비스 레이어 패턴

생성할 파일:
- wrangler.toml
- src/index.js (엔트리 포인트)
- src/middleware/errorHandler.js
- .gitignore
- package.json"
```

### 시나리오 2: .claude 폴더 설정

```
프롬프트: "이 Workers 프로젝트를 위한 .claude 폴더를 설정해줘.

파일:
- CLAUDE.md (프로젝트 컨텍스트)
- .claude/rules/architecture.md (아키텍처 규칙)
- .claude/templates/api-endpoint.md (API 템플릿)

프로젝트 정보:
- 기술 스택: Hono, D1, KV
- 패턴: 서비스 레이어, KV 캐시 우선"
```

---

## CRUD 개발

### 시나리오 3: 전체 CRUD 생성

```
프롬프트: "User CRUD를 완전히 만들어줘.

테이블 스키마 (D1):
- id INTEGER PRIMARY KEY AUTOINCREMENT
- email TEXT UNIQUE NOT NULL
- name TEXT NOT NULL
- password TEXT NOT NULL
- role TEXT DEFAULT 'user'
- created_at TEXT NOT NULL

생성할 파일:
1. migrations/0001_create_users.sql (D1 마이그레이션)
2. src/services/userService.js (비즈니스 로직)
3. src/routes/users.js (API 엔드포인트)
4. src/index.js에 라우트 등록

기능:
- GET /users - 목록 (페이징)
- GET /users/:id - 상세
- POST /users - 생성 (비밀번호 해싱)
- PUT /users/:id - 수정
- DELETE /users/:id - 삭제

규칙:
- KV 캐시 우선 사용 (TTL 1시간)
- bcrypt로 비밀번호 해싱
- 이메일 중복 체크"
```

### 시나리오 4: 목록 페이지 (페이징 + 검색)

```
프롬프트: "Product 목록 API에 페이징과 검색을 추가해줘.

요구사항:
- GET /products?page=1&limit=20&search=keyword&category=electronics
- 페이징: page, limit 파라미터
- 검색: name, description에서 검색 (대소문자 구분 없음)
- 필터: category 파라미터
- 응답: { data: [...], total, page, limit, totalPages }

정렬:
- 기본: created_at DESC
- ?sort=price_asc, price_desc, name_asc, name_desc"
```

---

## 인증 및 권한

### 시나리오 5: JWT 인증 구현

```
프롬프트: "JWT 기반 인증 시스템을 만들어줘.

엔드포인트:
- POST /auth/login - 로그인
- POST /auth/register - 회원가입
- GET /auth/profile - 내 정보 (인증 필요)
- POST /auth/refresh - 토큰 갱신

구현:
1. src/services/authService.js (로그인, 회원가입 로직)
2. src/middleware/auth.js (JWT 검증 미들웨어)
3. src/routes/auth.js (엔드포인트)

기능:
- bcrypt 비밀번호 해싱
- JWT 토큰 발급 (7일 만료)
- Refresh 토큰 (30일 만료)
- Rate Limiting (로그인 15분에 5번)
- 로그인 실패 로깅

환경 변수:
- JWT_SECRET
- JWT_REFRESH_SECRET"
```

### 시나리오 6: 역할 기반 권한

```
프롬프트: "역할 기반 권한 시스템을 추가해줘.

역할:
- admin: 모든 권한
- manager: 팀 관리
- user: 기본 권한

구현:
1. src/middleware/permission.js (권한 체크 미들웨어)
2. 기존 라우트에 권한 미들웨어 적용

함수:
- requireRole(role) - 특정 역할 필요
- requireAnyRole([roles]) - 여러 역할 중 하나
- isOwnerOrAdmin(resourceId) - 소유자 또는 관리자

적용:
- DELETE /users/:id → requireRole('admin')
- GET /admin/dashboard → requireRole('admin')
- PUT /products/:id → requireAnyRole(['admin', 'manager'])
- DELETE /posts/:id → isOwnerOrAdmin(postId)"
```

---

## 프론트엔드 개발

### 시나리오 7: ViewLogic 페이지

```
프롬프트: "제품 목록 페이지를 만들어줘. ViewLogic 패턴 사용.

파일:
- src/views/products/list.html (HTML 템플릿)
- src/logic/products/list.js (JavaScript 로직)

기능:
- 제품 목록 표시 (카드 형태)
- 카테고리별 필터
- 검색 기능
- 페이징
- 장바구니 담기 버튼

API:
- GET /api/products?page=1&limit=20&search=&category=

스타일:
- Bootstrap 5 클래스 사용
- CSS 변수 사용 (--primary-color 등)
- 반응형 (col-12 col-md-6 col-lg-4)
- HTML에 style 태그 금지"
```

### 시나리오 8: 폼 유효성 검증

```
프롬프트: "제품 등록 폼에 유효성 검증을 추가해줘.

폼 필드:
- name (필수, 2-100자)
- price (필수, 숫자, 0 이상)
- category (필수, 선택)
- description (선택, 1000자 이하)
- images (선택, 최대 5개, jpg/png만)

검증:
- 실시간 검증 (입력 중)
- 제출 시 전체 검증
- 에러 메시지 표시 (필드 아래)
- 포커스 이동 (첫 에러 필드)

구현:
- src/logic/products/create.js에 validateForm() 함수
- 에러 메시지는 한글로"
```

---

## 데이터베이스 작업

### 시나리오 9: 마이그레이션 생성

```
프롬프트: "다음 테이블들을 위한 D1 마이그레이션 파일을 만들어줘.

테이블:
1. categories
   - id INTEGER PRIMARY KEY
   - name TEXT UNIQUE NOT NULL
   - slug TEXT UNIQUE NOT NULL
   - created_at TEXT NOT NULL

2. products
   - id INTEGER PRIMARY KEY
   - category_id INTEGER NOT NULL
   - name TEXT NOT NULL
   - price INTEGER NOT NULL
   - stock INTEGER DEFAULT 0
   - created_at TEXT NOT NULL
   - FOREIGN KEY (category_id) REFERENCES categories(id)

3. orders
   - id INTEGER PRIMARY KEY
   - user_id INTEGER NOT NULL
   - total_amount INTEGER NOT NULL
   - status TEXT DEFAULT 'pending'
   - created_at TEXT NOT NULL

인덱스:
- products: category_id, created_at
- orders: user_id, status, created_at

파일: migrations/0001_initial_schema.sql"
```

### 시나리오 10: 복잡한 쿼리

```
프롬프트: "주문 통계 API를 만들어줘.

엔드포인트: GET /api/admin/stats/orders

쿼리:
- 오늘/이번 주/이번 달 주문 수
- 오늘/이번 주/이번 달 매출
- 상태별 주문 수 (pending, paid, shipped, delivered)
- 인기 제품 Top 10 (이번 달)
- 사용자별 주문 금액 Top 10

최적화:
- 단일 쿼리로 가능한 것은 JOIN 사용
- 결과 캐싱 (KV, TTL 10분)

구현:
- src/services/statsService.js
- src/routes/admin/stats.js"
```

---

## 성능 최적화

### 시나리오 11: N+1 문제 해결

```
프롬프트: "이 코드에 N+1 쿼리 문제가 있어. 최적화해줘.

현재 코드:
const posts = await db.query('SELECT * FROM posts');
for (const post of posts) {
  post.author = await db.query('SELECT * FROM users WHERE id = ?', [post.user_id]);
  post.comments = await db.query('SELECT * FROM comments WHERE post_id = ?', [post.id]);
}

최적화:
- JOIN 사용
- 단일 쿼리로 변경
- 결과를 적절히 그룹화"
```

### 시나리오 12: 캐싱 추가

```
프롬프트: "UserService에 KV 캐싱을 추가해줘.

대상 메소드:
- getUser(id) → 캐시 키: user:{id}, TTL 1시간
- listUsers() → 캐시 키: users:list, TTL 5분
- getUserByEmail(email) → 캐시 키: user:email:{email}, TTL 1시간

캐시 무효화:
- createUser() → users:list 무효화
- updateUser(id) → user:{id}, user:email:{email} 무효화
- deleteUser(id) → user:{id}, user:email:{email}, users:list 무효화

패턴:
- Cache-Aside (캐시 우선 조회)
- 캐시 없으면 DB 조회 후 캐시 저장"
```

---

## 테스트 작성

### 시나리오 13: 단위 테스트

```
프롬프트: "UserService의 모든 메소드에 대한 단위 테스트를 작성해줘.

테스트 프레임워크: Jest

테스트 케이스:
1. createUser()
   - 정상 생성
   - 이메일 중복 에러
   - 이메일 형식 오류
   - 필수 필드 누락

2. getUser()
   - 정상 조회
   - 존재하지 않는 사용자
   - 캐시 히트 확인

3. updateUser()
   - 정상 수정
   - 존재하지 않는 사용자
   - 캐시 무효화 확인

Mock:
- DB는 모킹
- KV는 모킹
- 환경 변수 모킹

파일: src/services/userService.test.js"
```

### 시나리오 14: 통합 테스트

```
프롬프트: "User API의 전체 플로우를 테스트해줘.

시나리오:
1. 회원가입 (POST /auth/register)
2. 로그인 (POST /auth/login) → JWT 토큰 획득
3. 프로필 조회 (GET /auth/profile) with JWT
4. 프로필 수정 (PUT /users/:id) with JWT
5. 로그아웃 (토큰 무효화)

검증:
- 각 단계의 HTTP 상태 코드
- 응답 데이터 구조
- 에러 케이스 (인증 실패, 권한 없음)

파일: tests/integration/user.test.js"
```

---

## 리팩토링 및 개선

### 시나리오 15: 코드 리팩토링

```
프롬프트: "이 파일을 리팩토링해줘: src/services/orderService.js

문제점:
- 함수가 너무 길다 (createOrder 함수 200줄)
- 중복 코드가 많다
- 에러 처리가 일관적이지 않다
- 변수명이 모호하다

개선:
- 큰 함수를 작은 함수로 분리
- DRY 원칙 적용
- 일관된 에러 처리 (커스텀 에러 클래스)
- 명확한 변수명 사용
- JSDoc 주석 추가"
```

### 시나리오 16: 보안 강화

```
프롬프트: "이 API의 보안을 강화해줘: src/routes/users.js

추가할 보안 기능:
- Rate Limiting (15분에 100 요청)
- 입력 검증 (Zod 스키마)
- SQL Injection 방지 확인
- XSS 방지 확인
- CSRF 토큰 (폼 제출 시)
- 보안 헤더 추가
- 로깅 (실패한 인증 시도)

검토:
- 현재 취약점 찾기
- 권장 수정사항 제시"
```

---

## 문제 해결

### 시나리오 17: 버그 수정

```
프롬프트: "이 에러를 수정해줘.

에러 메시지:
TypeError: Cannot read property 'id' of undefined
at ProductService.getProduct (productService.js:45)

발생 조건:
- GET /products/999 (존재하지 않는 ID)
- 매번 발생

현재 코드:
async getProduct(id) {
  const cached = await this.env.KV.get(`product:${id}`, { type: 'json' });
  if (cached) return cached;

  const product = await this.env.DB
    .prepare('SELECT * FROM products WHERE id = ?')
    .bind(id)
    .first();

  await this.env.KV.put(`product:${id}`, JSON.stringify(product), {
    expirationTtl: 3600
  });

  return product;
}

수정 요청:
- null 체크 추가
- NotFoundError 발생
- 에러 로깅"
```

### 시나리오 18: 성능 문제 진단

```
프롬프트: "이 API가 너무 느려. 원인을 찾아서 개선해줘.

API: GET /api/dashboard
응답 시간: 5-10초 (목표: 1초 이내)

현재 코드: src/services/dashboardService.js

분석 요청:
1. 병목 지점 찾기
2. N+1 쿼리 확인
3. 불필요한 순차 실행 확인
4. 캐싱 기회 확인

개선 제안:
- 구체적인 수정 방안
- 예상 성능 향상"
```

---

## 다음 단계

- [효과적인 프롬프트 작성 팁](effective-prompts.md) - 고급 테크닉
- [자주 사용하는 프롬프트 패턴](prompt-patterns.md) - 기본 패턴
- [컨텍스트 관리 및 체크포인트](context-management.md) - 장기 프로젝트

---

[← 목차로 돌아가기](../_sidebar.md)
