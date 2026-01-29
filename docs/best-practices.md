# 베스트 프랙티스

## 개요

Claude Code를 사용한 개발에서 **검증된 패턴과 권장 사항**을 제시합니다.

## 프로젝트 구조

### 1. 레이어 분리

**명확한 책임 분리:**
```
routes/     → 요청 처리, 응답 반환
services/   → 비즈니스 로직
utils/      → 공통 유틸리티
middleware/ → 횡단 관심사 (인증, 로깅)
```

**예제:**
```javascript
// ✅ 좋은 예: 레이어 분리
// routes/users.js
users.post('/', async (c) => {
  const data = await c.req.json();
  const service = new UserService(c.env);
  const user = await service.createUser(data);
  return c.json({ data: user }, 201);
});

// services/userService.js
class UserService {
  async createUser(data) {
    // 검증
    this.validateUser(data);

    // 저장
    const user = await this.saveToDb(data);

    // 이메일 발송
    await this.sendWelcomeEmail(user.email);

    return user;
  }
}
```

### 2. 설정 중앙화

**config.js로 통합:**
```javascript
// ✅ 중앙 설정
// config/index.js
export const config = {
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: '7d'
  },
  cache: {
    ttl: 3600,
    prefix: 'app:'
  },
  rateLimit: {
    windowMs: 15 * 60 * 1000,
    max: 100
  }
};

// 사용
import { config } from './config';
const token = jwt.sign(payload, config.jwt.secret, {
  expiresIn: config.jwt.expiresIn
});
```

## 코드 품질

### 1. DRY (Don't Repeat Yourself)

**중복 코드 제거:**
```javascript
// ❌ 중복
function getUserById(id) {
  return db.prepare('SELECT * FROM users WHERE id = ?').bind(id).first();
}

function getProductById(id) {
  return db.prepare('SELECT * FROM products WHERE id = ?').bind(id).first();
}

// ✅ 공통 함수 추출
function findById(table, id) {
  return db.prepare(`SELECT * FROM ${table} WHERE id = ?`).bind(id).first();
}

const user = await findById('users', userId);
const product = await findById('products', productId);
```

### 2. 조기 반환 (Early Return)

**중첩 감소:**
```javascript
// ❌ 중첩이 많음
function processUser(user) {
  if (user) {
    if (user.email) {
      if (user.isActive) {
        // 처리
        return user;
      } else {
        throw new Error('Inactive user');
      }
    } else {
      throw new Error('Email required');
    }
  } else {
    throw new Error('User not found');
  }
}

// ✅ 조기 반환
function processUser(user) {
  if (!user) throw new Error('User not found');
  if (!user.email) throw new Error('Email required');
  if (!user.isActive) throw new Error('Inactive user');

  // 처리
  return user;
}
```

### 3. 함수형 프로그래밍

**선언적 코드:**
```javascript
// ❌ 명령형
const activeUsers = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].isActive) {
    activeUsers.push(users[i]);
  }
}

// ✅ 선언형
const activeUsers = users.filter(user => user.isActive);

// ✅ 체이닝
const result = users
  .filter(user => user.isActive)
  .map(user => user.name)
  .sort();
```

## 성능 최적화

### 1. 데이터베이스 쿼리

**N+1 문제 해결:**
```javascript
// ❌ N+1 문제
const users = await db.query('SELECT * FROM users');
for (const user of users) {
  const orders = await db.query('SELECT * FROM orders WHERE user_id = ?', [user.id]);
  user.orders = orders;
}

// ✅ JOIN 사용
const users = await db.query(`
  SELECT u.*, o.*
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
`);
```

**인덱스 활용:**
```sql
-- ✅ 자주 조회하는 컬럼에 인덱스
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_created_at ON orders(created_at);
```

### 2. 캐싱 전략

**Cache-Aside 패턴:**
```javascript
// ✅ 캐시 우선 조회
async function getUser(id) {
  // 1. 캐시 확인
  const cacheKey = `user:${id}`;
  const cached = await cache.get(cacheKey);
  if (cached) return JSON.parse(cached);

  // 2. DB 조회
  const user = await db.findUser(id);

  // 3. 캐시 저장
  if (user) {
    await cache.set(cacheKey, JSON.stringify(user), { ttl: 3600 });
  }

  return user;
}
```

**캐시 무효화:**
```javascript
// ✅ 업데이트 시 캐시 무효화
async function updateUser(id, data) {
  const user = await db.updateUser(id, data);

  // 캐시 무효화
  await cache.del(`user:${id}`);

  return user;
}
```

### 3. 병렬 처리

**Promise.all 활용:**
```javascript
// ❌ 순차 실행 (느림)
const user = await getUser(userId);
const orders = await getOrders(userId);
const reviews = await getReviews(userId);

// ✅ 병렬 실행 (빠름)
const [user, orders, reviews] = await Promise.all([
  getUser(userId),
  getOrders(userId),
  getReviews(userId)
]);
```

## 에러 처리

### 1. 의미 있는 에러

**구체적인 에러 메시지:**
```javascript
// ❌ 모호한 에러
if (!user) throw new Error('Error');

// ✅ 구체적인 에러
if (!user) throw new NotFoundError(`User not found: ${userId}`);
if (!user.email) throw new ValidationError('Email is required');
if (!user.isActive) throw new ForbiddenError('User account is inactive');
```

### 2. 전역 에러 핸들러

**일관된 에러 응답:**
```javascript
// middleware/errorHandler.js
export function errorHandler(error, c) {
  // 로깅
  console.error('Error:', error);

  // 에러 타입별 처리
  if (error.name === 'ValidationError') {
    return c.json({ error: error.message }, 400);
  }

  if (error.name === 'NotFoundError') {
    return c.json({ error: error.message }, 404);
  }

  if (error.name === 'UnauthorizedError') {
    return c.json({ error: 'Unauthorized' }, 401);
  }

  // 기본 에러
  return c.json({ error: 'Internal server error' }, 500);
}

// 사용
app.onError(errorHandler);
```

## 테스트

### 1. 단위 테스트

**작은 단위로 테스트:**
```javascript
// services/userService.test.js
describe('UserService', () => {
  it('should create user with valid data', async () => {
    const service = new UserService(mockEnv);
    const user = await service.createUser({
      email: 'test@example.com',
      name: 'Test User'
    });

    expect(user.email).toBe('test@example.com');
    expect(user.id).toBeDefined();
  });

  it('should throw error with invalid email', async () => {
    const service = new UserService(mockEnv);

    await expect(
      service.createUser({ email: 'invalid', name: 'Test' })
    ).rejects.toThrow('Invalid email format');
  });
});
```

### 2. 통합 테스트

**API 엔드포인트 테스트:**
```javascript
describe('POST /api/users', () => {
  it('should create user', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({
        email: 'test@example.com',
        name: 'Test User'
      });

    expect(response.status).toBe(201);
    expect(response.body.data).toHaveProperty('id');
  });
});
```

## 문서화

### 1. 코드 주석

**JSDoc 활용:**
```javascript
/**
 * 사용자를 생성합니다.
 *
 * @param {Object} data - 사용자 데이터
 * @param {string} data.email - 이메일 (필수)
 * @param {string} data.name - 이름 (필수)
 * @returns {Promise<User>} 생성된 사용자
 * @throws {ValidationError} 검증 실패 시
 */
async function createUser(data) {
  // ...
}
```

### 2. API 문서

**OpenAPI/Swagger:**
```yaml
paths:
  /api/users:
    post:
      summary: 사용자 생성
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, name]
              properties:
                email:
                  type: string
                  format: email
                name:
                  type: string
      responses:
        '201':
          description: 생성 성공
        '400':
          description: 검증 실패
```

## Claude Code 활용 패턴

### 1. 리팩토링 요청

```
프롬프트: "이 함수를 리팩토링해줘.

개선사항:
- DRY 원칙 적용
- 조기 반환으로 중첩 감소
- 의미 있는 변수명"
```

### 2. 테스트 생성

```
프롬프트: "UserService의 모든 메소드에 대한 단위 테스트를 작성해줘.
Jest 프레임워크 사용."
```

### 3. 성능 개선

```
프롬프트: "이 코드의 성능을 개선해줘.

체크 항목:
- N+1 쿼리 문제
- 불필요한 순차 실행
- 캐싱 기회"
```

---

## 체크리스트

새 기능 개발 시:

**코드 품질:**
- [ ] 레이어 분리 (routes, services, utils)
- [ ] DRY 원칙 적용
- [ ] 조기 반환으로 중첩 최소화
- [ ] 의미 있는 변수/함수명

**성능:**
- [ ] DB 쿼리 최적화 (인덱스, JOIN)
- [ ] 캐싱 적용 (KV, 메모리)
- [ ] 병렬 처리 (Promise.all)

**에러 처리:**
- [ ] 구체적인 에러 메시지
- [ ] 에러 타입 구분
- [ ] 전역 에러 핸들러

**테스트:**
- [ ] 단위 테스트 작성
- [ ] 엣지 케이스 커버
- [ ] 통합 테스트

**문서화:**
- [ ] 함수 주석 (JSDoc)
- [ ] README 업데이트
- [ ] API 문서 (선택)

---

## 다음 단계

- [코드 리뷰 체크리스트](code-review-checklist.md) - 리뷰 항목
- [코딩 규칙](coding-rules.md) - 기본 규칙
- [보안 가이드라인](security.md) - 보안 규칙

---

[← 목차로 돌아가기](../_sidebar.md)
