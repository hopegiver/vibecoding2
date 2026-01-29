# 보안 가이드라인

## 개요

Claude Code를 사용할 때도 **보안은 최우선**입니다. AI가 생성한 코드에서 흔히 발생하는 보안 취약점을 이해하고 방지하세요.

## OWASP Top 10 체크리스트

### 1. Injection 방지

**SQL Injection:**
```javascript
// ❌ 위험: 문자열 결합
const query = `SELECT * FROM users WHERE id = ${userId}`;
const user = await db.query(query);

// ✅ 안전: Prepared Statement
const user = await db.prepare('SELECT * FROM users WHERE id = ?')
  .bind(userId)
  .first();
```

**Command Injection:**
```javascript
// ❌ 위험: 사용자 입력 직접 실행
const { exec } = require('child_process');
exec(`ping ${userInput}`);

// ✅ 안전: 입력 검증 + 화이트리스트
const allowedHosts = ['localhost', 'example.com'];
if (allowedHosts.includes(userInput)) {
  exec(`ping ${userInput}`);
}
```

### 2. 인증 및 세션 관리

**JWT 보안:**
```javascript
// ✅ 강력한 시크릿 키 사용
const JWT_SECRET = process.env.JWT_SECRET; // 최소 32자 이상

// ✅ 토큰 만료 설정
const token = jwt.sign(
  { userId: user.id },
  JWT_SECRET,
  { expiresIn: '7d' }  // 7일 후 만료
);

// ❌ 절대 금지: 시크릿 키 하드코딩
const token = jwt.sign(payload, 'my-secret-key');
```

**비밀번호 해싱:**
```javascript
// ✅ bcrypt 사용
import bcrypt from 'bcryptjs';

// 저장 시
const hashedPassword = await bcrypt.hash(password, 10);

// 검증 시
const isValid = await bcrypt.compare(inputPassword, hashedPassword);

// ❌ 절대 금지: 평문 저장
user.password = password; // 위험!
```

### 3. XSS (Cross-Site Scripting) 방지

**출력 인코딩:**
```javascript
// ✅ 프레임워크 기본 이스케이프 사용
// Vue.js
<div>{{ userInput }}</div>  // 자동 이스케이프

// React
<div>{userInput}</div>  // 자동 이스케이프

// ❌ 위험: 직접 HTML 삽입
<div v-html="userInput"></div>
<div dangerouslySetInnerHTML={{__html: userInput}}></div>
```

**맑은프레임워크 XSS 방지:**
```jsp
<!-- ✅ GET 파라미터: m.rs() 사용 (자동 XSS 필터) -->
<%
String keyword = m.rs("keyword");  // XSS 필터 자동 적용
%>

<!-- ❌ 위험: request.getParameter() 직접 사용 -->
<%
String keyword = request.getParameter("keyword");  // XSS 필터 없음!
%>
```

### 4. CSRF (Cross-Site Request Forgery) 방지

**CSRF 토큰 사용:**
```html
<!-- ✅ 폼에 CSRF 토큰 포함 -->
<form method="post">
  <input type="hidden" name="_csrf" value="{{ csrfToken }}">
  <button type="submit">제출</button>
</form>
```

```javascript
// 서버에서 검증
app.post('/api/users', csrfMiddleware, async (c) => {
  // CSRF 토큰 검증 후 처리
});
```

**SameSite 쿠키:**
```javascript
// ✅ SameSite 속성 설정
c.cookie('token', jwt, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict'  // CSRF 방지
});
```

### 5. 인가(Authorization) 체크

**권한 확인:**
```javascript
// ✅ 모든 요청에서 권한 체크
app.delete('/api/users/:id', authMiddleware, async (c) => {
  const userId = c.get('userId');  // JWT에서 추출
  const targetId = c.req.param('id');

  // 본인 또는 관리자만 삭제 가능
  if (userId !== targetId && c.get('userRole') !== 'admin') {
    return c.json({ error: 'Forbidden' }, 403);
  }

  // 삭제 진행
});

// ❌ 위험: 권한 체크 없음
app.delete('/api/users/:id', async (c) => {
  const id = c.req.param('id');
  await deleteUser(id);  // 누구나 삭제 가능!
});
```

### 6. 민감 데이터 노출 방지

**환경 변수 사용:**
```javascript
// ✅ 환경 변수에서 읽기
const API_KEY = process.env.API_KEY;
const DB_PASSWORD = process.env.DB_PASSWORD;

// ❌ 절대 금지: 하드코딩
const API_KEY = 'sk_live_12345...';
const DB_PASSWORD = 'mypassword123';
```

**.env 파일 관리:**
```bash
# .gitignore에 반드시 추가
.env
.env.local
*.key
*.pem
```

**민감 정보 로그 제거:**
```javascript
// ❌ 위험: 비밀번호 로깅
console.log('User login:', { email, password });

// ✅ 안전: 민감 정보 제외
console.log('User login:', { email });
```

### 7. 보안 설정 오류 방지

**CORS 설정:**
```javascript
// ✅ 특정 도메인만 허용
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com']
}));

// ❌ 위험: 모든 도메인 허용
app.use(cors({
  origin: '*'  // 프로덕션에서 금지!
}));
```

**보안 헤더 설정:**
```javascript
// ✅ 보안 헤더 추가
app.use(async (c, next) => {
  c.header('X-Content-Type-Options', 'nosniff');
  c.header('X-Frame-Options', 'DENY');
  c.header('X-XSS-Protection', '1; mode=block');
  c.header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
  await next();
});
```

### 8. 파일 업로드 보안

**파일 타입 검증:**
```javascript
// ✅ MIME 타입 및 확장자 검증
const allowedTypes = ['image/jpeg', 'image/png', 'image/gif'];
const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif'];

if (!allowedTypes.includes(file.type)) {
  throw new Error('Invalid file type');
}

const ext = path.extname(file.name).toLowerCase();
if (!allowedExtensions.includes(ext)) {
  throw new Error('Invalid file extension');
}

// ❌ 위험: 확장자만 검증
if (filename.endsWith('.jpg')) {
  // 우회 가능: file.php.jpg
}
```

**파일 크기 제한:**
```javascript
// ✅ 크기 제한
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

if (file.size > MAX_FILE_SIZE) {
  throw new Error('File too large');
}
```

### 9. 로깅 및 모니터링

**보안 이벤트 로깅:**
```javascript
// ✅ 중요 이벤트 로깅
function logSecurityEvent(event, details) {
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    event,
    details,
    severity: 'WARNING'
  }));
}

// 로그인 실패
logSecurityEvent('LOGIN_FAILED', { email, ip: c.req.header('CF-Connecting-IP') });

// 권한 없는 접근 시도
logSecurityEvent('UNAUTHORIZED_ACCESS', { userId, resource, ip });
```

### 10. Rate Limiting

**요청 제한:**
```javascript
// ✅ Rate Limiting 적용
import { rateLimiter } from 'hono-rate-limiter';

app.use('/api/*', rateLimiter({
  windowMs: 15 * 60 * 1000, // 15분
  max: 100, // 최대 100 요청
  message: 'Too many requests'
}));

// 로그인은 더 엄격하게
app.use('/api/auth/login', rateLimiter({
  windowMs: 15 * 60 * 1000,
  max: 5  // 15분에 5번만
}));
```

---

## Claude Code 사용 시 보안 체크

### 1. 생성된 코드 보안 검토

Claude Code가 코드를 생성하면 다음을 확인하세요:

```
프롬프트: "방금 생성한 코드의 보안 이슈를 체크해줘.

체크 항목:
- SQL Injection 가능성
- XSS 취약점
- 하드코딩된 시크릿
- 권한 체크 누락
- 입력 검증 누락"
```

### 2. 보안 규칙 명시

프롬프트에 보안 요구사항 포함:

```
프롬프트: "로그인 API를 만들어줘.

보안 요구사항:
- bcrypt로 비밀번호 해싱
- JWT 토큰 발급 (7일 만료)
- Rate Limiting (15분에 5번)
- 실패 시 로깅"
```

### 3. .env 파일 생성 요청

```
프롬프트: "API 키와 DB 비밀번호를 환경 변수로 설정해줘.
.env.example 파일도 만들어줘."
```

---

## 보안 체크리스트

새 기능 개발 시:

- [ ] **입력 검증**: 모든 사용자 입력 검증
- [ ] **SQL Injection**: Prepared Statement 사용
- [ ] **XSS**: 출력 이스케이프
- [ ] **인증**: JWT 또는 세션 검증
- [ ] **인가**: 권한 체크
- [ ] **CSRF**: CSRF 토큰 또는 SameSite 쿠키
- [ ] **환경 변수**: 시크릿은 .env에
- [ ] **Rate Limiting**: API 요청 제한
- [ ] **로깅**: 보안 이벤트 기록
- [ ] **CORS**: 허용 도메인 제한

---

## 보안 사고 대응

### 취약점 발견 시

1. **즉시 수정**: 취약점 발견 시 최우선 수정
2. **영향 범위 파악**: 어떤 데이터가 노출되었는가?
3. **로그 확인**: 공격 시도가 있었는가?
4. **사용자 알림**: 필요 시 사용자에게 통지
5. **재발 방지**: 같은 패턴의 취약점 전체 검토

### Claude Code 활용

```
프롬프트: "우리 코드베이스에서 SQL Injection 취약점이 있는 곳을 찾아줘.
모든 DB 쿼리를 검토해줘."
```

---

## 다음 단계

- [베스트 프랙티스](best-practices.md) - 권장 패턴
- [코드 리뷰 체크리스트](code-review-checklist.md) - 리뷰 항목
- [코딩 규칙](coding-rules.md) - 기본 규칙

---

[← 목차로 돌아가기](../_sidebar.md)
