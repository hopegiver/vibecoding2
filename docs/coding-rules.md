# 코딩 규칙

## 개요

Claude Code를 사용할 때도 **기본적인 코딩 규칙**을 준수해야 합니다. AI가 생성한 코드라도 반드시 검토하고, 프로젝트 표준을 따르는지 확인하세요.

## 공통 규칙

모든 플랫폼에 공통으로 적용되는 규칙입니다.

### 1. 명명 규칙

**변수명:**
```javascript
// ✅ 좋은 예
const userName = 'John';
const isAuthenticated = true;
const maxRetryCount = 3;

// ❌ 나쁜 예
const un = 'John';
const flag = true;
const max = 3;
```

**함수명:**
```javascript
// ✅ 동사로 시작
function getUserById(id) { }
function validateEmail(email) { }
function createOrder(data) { }

// ❌ 명사만 사용
function user(id) { }
function email(email) { }
```

**클래스명:**
```javascript
// ✅ PascalCase
class UserService { }
class OrderManager { }

// ❌ camelCase
class userService { }
```

### 2. 함수 작성 규칙

**함수는 하나의 일만:**
```javascript
// ❌ 여러 일을 하는 함수
function processUser(user) {
  // 검증
  if (!user.email) throw new Error('Email required');

  // DB 저장
  await db.insert('users', user);

  // 이메일 발송
  await sendEmail(user.email, 'Welcome!');

  // 로그
  console.log('User created:', user.id);
}

// ✅ 분리된 함수들
function validateUser(user) {
  if (!user.email) throw new Error('Email required');
  return true;
}

async function saveUser(user) {
  return await db.insert('users', user);
}

async function sendWelcomeEmail(email) {
  return await sendEmail(email, 'Welcome!');
}

function logUserCreation(userId) {
  console.log('User created:', userId);
}
```

**함수 길이:**
- 최대 50줄 이내
- 화면 한 페이지에 들어올 정도
- 너무 길면 작은 함수들로 분리

### 3. 주석 규칙

**좋은 주석:**
```javascript
// ✅ 왜(Why)를 설명
// 캐시를 1시간으로 설정: API 호출 비용 절감을 위해
const CACHE_TTL = 3600;

// 3번 재시도: 네트워크 일시적 오류 대응
const MAX_RETRIES = 3;
```

**나쁜 주석:**
```javascript
// ❌ 무엇(What)을 반복
// 사용자 이름을 가져옴
const name = user.name;

// 카운트를 1 증가
count++;
```

**코드로 설명:**
```javascript
// ❌ 주석으로 설명
// 사용자가 관리자이거나 소유자인 경우
if (user.role === 'admin' || user.id === resource.ownerId) {
  // 허용
}

// ✅ 함수명으로 설명
function canAccessResource(user, resource) {
  return user.role === 'admin' || user.id === resource.ownerId;
}

if (canAccessResource(user, resource)) {
  // 허용
}
```

### 4. 에러 처리

**명확한 에러 메시지:**
```javascript
// ❌ 모호한 에러
throw new Error('Error');
throw new Error('Invalid input');

// ✅ 구체적인 에러
throw new Error('User not found: ID ' + userId);
throw new Error('Email format invalid: ' + email);
```

**에러 타입 구분:**
```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

class NotFoundError extends Error {
  constructor(message) {
    super(message);
    this.name = 'NotFoundError';
  }
}

// 사용
if (!user.email) {
  throw new ValidationError('Email is required');
}

if (!foundUser) {
  throw new NotFoundError(`User not found: ${userId}`);
}
```

### 5. 비동기 처리

**async/await 사용:**
```javascript
// ✅ async/await
async function getUser(id) {
  try {
    const user = await db.findUser(id);
    return user;
  } catch (error) {
    console.error('Failed to get user:', error);
    throw error;
  }
}

// ❌ Promise then/catch
function getUser(id) {
  return db.findUser(id)
    .then(user => user)
    .catch(error => {
      console.error('Failed to get user:', error);
      throw error;
    });
}
```

**병렬 처리:**
```javascript
// ✅ 병렬 실행
const [users, products, orders] = await Promise.all([
  getUsers(),
  getProducts(),
  getOrders()
]);

// ❌ 순차 실행 (불필요)
const users = await getUsers();
const products = await getProducts();
const orders = await getOrders();
```

---

## 플랫폼별 규칙

### Workers 규칙

**1. 서비스 레이어 필수:**
```javascript
// ✅ 서비스 레이어 사용
// routes/users.js
users.get('/:id', async (c) => {
  const service = new UserService(c.env);
  const user = await service.getUser(c.req.param('id'));
  return c.json({ data: user });
});

// ❌ 라우트에서 직접 DB 접근
users.get('/:id', async (c) => {
  const user = await c.env.DB.prepare('SELECT * FROM users WHERE id = ?')
    .bind(c.req.param('id'))
    .first();
  return c.json({ data: user });
});
```

**2. KV 캐시 활용:**
```javascript
// ✅ KV 캐시 우선
async getUser(id) {
  // 1. 캐시 확인
  const cached = await this.env.KV.get(`user:${id}`, { type: 'json' });
  if (cached) return cached;

  // 2. DB 조회
  const user = await this.env.DB.prepare('SELECT * FROM users WHERE id = ?').bind(id).first();

  // 3. 캐시 저장
  if (user) {
    await this.env.KV.put(`user:${id}`, JSON.stringify(user), { expirationTtl: 3600 });
  }

  return user;
}
```

**3. 환경 변수 주입:**
```javascript
// ✅ 생성자에서 env 주입
class UserService {
  constructor(env) {
    this.env = env;
  }
}

// ❌ 전역 변수 사용
let globalEnv;
class UserService {
  getUser() {
    return globalEnv.DB.prepare('...');
  }
}
```

### Pages (ViewLogic) 규칙

**1. HTML/JS 완전 분리:**
```javascript
// ✅ logic/goals/my-goals.js
export default {
  layout: 'default',
  data() {
    return {
      goals: []
    }
  },
  async mounted() {
    this.goals = await this.$api.get('/api/goals');
  }
}
```

```html
<!-- ✅ views/goals/my-goals.html -->
<div v-for="goal in goals" :key="goal.id">
  <h3>{{ goal.title }}</h3>
</div>
```

```html
<!-- ❌ HTML에 스타일 금지 -->
<style>
.goal { color: red; }
</style>
```

**2. CSS 변수 사용:**
```css
/* ✅ CSS 변수 */
.primary-button {
  background: var(--primary-color);
  color: white;
}

/* ❌ 하드코딩 */
.primary-button {
  background: #6366f1;
  color: white;
}
```

### 맑은프레임워크 규칙

**1. JSP/HTML 분리:**
```jsp
<!-- ✅ JSP: 로직만 -->
<%
UserDao user = new UserDao();
DataSet list = user.find();

p.setBody("main.user_list");
p.setLoop("users", list);
p.display();
%>
```

```html
<!-- ✅ HTML: 템플릿만 -->
<!--@loop(users)-->
<div>{{users.name}}</div>
<!--/loop(users)-->
```

**2. try-catch 금지:**
```jsp
<!-- ✅ 프레임워크 메소드 활용 -->
<%
if(user.insert()) {
    m.jsAlert("성공");
} else {
    m.jsAlert(user.getErrMsg());
}
%>

<!-- ❌ try-catch -->
<%
try {
    user.insert();
} catch(Exception e) {
    // 금지!
}
%>
```

**3. POST 후 return:**
```jsp
<!-- ✅ return 필수 -->
<%
if(m.isPost()) {
    user.insert();
    m.jsReplace("list.jsp");
    return;  // 필수!
}
p.display();
%>
```

---

## Claude Code 활용 시 주의사항

### 1. 생성된 코드 검토 필수

Claude Code가 생성한 코드도 **반드시 검토**하세요:
- 변수명이 명확한가?
- 함수가 너무 길지 않은가?
- 에러 처리가 적절한가?
- 보안 이슈는 없는가?

### 2. 프로젝트 규칙 명시

프롬프트에 규칙을 포함하세요:

```
프롬프트: "User CRUD API를 만들어줘.

규칙:
- 서비스 레이어 필수
- KV 캐시 우선 사용
- 함수는 50줄 이내
- async/await 사용"
```

### 3. 리팩토링 요청

코드가 규칙에 맞지 않으면 리팩토링 요청:

```
프롬프트: "이 함수가 너무 길어.
작은 함수들로 분리해줘. 각 함수는 하나의 일만 하도록."
```

### 4. 코드 리뷰 요청

```
프롬프트: "방금 생성한 코드를 리뷰해줘.

체크 항목:
- 명명 규칙 준수
- 함수 길이
- 에러 처리
- 보안 이슈"
```

---

## 다음 단계

- [보안 가이드라인](security.md) - 보안 규칙
- [베스트 프랙티스](best-practices.md) - 권장 패턴
- [코드 리뷰 체크리스트](code-review-checklist.md) - 리뷰 항목

---

[← 목차로 돌아가기](../_sidebar.md)
