# 문제 해결 - 성능 최적화

프로젝트 성능을 향상시키는 방법을 학습합니다.

## 데이터베이스 최적화

### 인덱스 추가

```sql
-- 자주 조회되는 컬럼에 인덱스
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at);

-- 복합 인덱스
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);
```

### 쿼리 최적화

**AS-IS (느림):**
```typescript
// N+1 문제
const posts = await env.DB.prepare('SELECT * FROM posts').all();
for (const post of posts.results) {
    const user = await env.DB.prepare('SELECT * FROM users WHERE id = ?').bind(post.user_id).first();
}
```

**TO-BE (빠름):**
```typescript
// JOIN 사용
const posts = await env.DB.prepare(`
    SELECT p.*, u.name as author_name
    FROM posts p
    JOIN users u ON p.user_id = u.id
`).all();
```

## 캐싱 전략

### KV 캐싱

```typescript
async function getCachedPost(id: string, env: Env) {
    // 캐시 확인
    const cached = await env.CACHE.get(`post:${id}`);
    if (cached) {
        return JSON.parse(cached);
    }

    // DB 조회
    const post = await env.DB.prepare('SELECT * FROM posts WHERE id = ?').bind(id).first();

    // 캐시 저장 (1시간)
    await env.CACHE.put(`post:${id}`, JSON.stringify(post), {
        expirationTtl: 3600
    });

    return post;
}
```

### HTTP 캐싱

```typescript
// _headers 파일
/*
  Cache-Control: public, max-age=3600
*/

// 또는 코드에서
return new Response(JSON.stringify(data), {
    headers: {
        'Content-Type': 'application/json',
        'Cache-Control': 'public, max-age=3600'
    }
});
```

## 프론트엔드 최적화

### 이미지 최적화

```html
<!-- 반응형 이미지 -->
<picture>
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="Image" loading="lazy" width="800" height="600">
</picture>
```

### 코드 스플리팅

```typescript
// 동적 import
const router = await import('./router.js');
```

### CSS 최적화

```css
/* Critical CSS 인라인 */
/* 나머지 CSS는 비동기 로드 */
```

## API 성능

### 페이징

```typescript
// 항상 LIMIT 사용
const { results } = await env.DB.prepare(
    'SELECT * FROM posts ORDER BY created_at DESC LIMIT ?'
).bind(limit).all();
```

### 필드 선택

```typescript
// 필요한 필드만 조회
const { results } = await env.DB.prepare(
    'SELECT id, title, excerpt FROM posts'  // content 제외
).all();
```

## Workers 최적화

### 불필요한 연산 제거

```typescript
// ❌ 느림
function processData(data: any[]) {
    return data.map(item => {
        // 복잡한 연산
        return heavyComputation(item);
    });
}

// ✅ 빠름
function processData(data: any[]) {
    // 필요한 것만 처리
    return data.map(item => ({ id: item.id, name: item.name }));
}
```

### 병렬 처리

```typescript
// ❌ 순차 처리 (느림)
const user = await fetchUser(id);
const posts = await fetchPosts(id);
const comments = await fetchComments(id);

// ✅ 병렬 처리 (빠름)
const [user, posts, comments] = await Promise.all([
    fetchUser(id),
    fetchPosts(id),
    fetchComments(id)
]);
```

## 실전 프롬프트 예시

### 성능 분석

```
다음 API의 성능을 분석해줘:

GET /api/posts

확인:
1. 쿼리 최적화 (EXPLAIN 사용)
2. N+1 문제
3. 인덱스 누락
4. 캐싱 적용 가능 여부
```

### 최적화 적용

```
게시판 목록 조회를 최적화해줘.

현재 문제:
- 100개 게시글 조회 시 2초 소요
- 각 게시글마다 작성자 정보 조회

개선:
1. JOIN으로 한 번에 조회
2. 필요한 인덱스 추가
3. KV 캐싱 적용
```

## 측정 도구

### Lighthouse

```bash
# 웹 성능 측정
lighthouse https://myapp.com --view
```

### Wrangler

```bash
# Workers 로그 확인
npx wrangler tail

# CPU 시간 확인
```

## 체크리스트

성능 최적화 확인사항:

- [ ] 데이터베이스 인덱스가 있는가?
- [ ] N+1 문제가 없는가?
- [ ] 캐싱을 적용했는가?
- [ ] 이미지를 최적화했는가?
- [ ] 불필요한 연산을 제거했는가?
- [ ] 병렬 처리를 했는가?

## 관련 문서

- [일반적인 오류 해결](troubleshooting.md)
- [디버깅 전략](debugging.md)
