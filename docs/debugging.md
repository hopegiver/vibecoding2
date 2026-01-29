# 문제 해결 - 디버깅 전략

효과적인 디버깅 방법과 도구 사용법을 학습합니다.

## Claude Code 디버깅

### 로그 출력

```
다음 함수에 로그를 추가해줘:

src/api.ts의 fetchPosts 함수

로그 위치:
- 함수 시작
- API 호출 전/후
- 에러 발생 시
```

### 단계별 실행

```
fetchPosts 함수를 단계별로 실행해줘:

1. 파라미터 출력
2. API 호출
3. 응답 확인
4. 결과 반환

각 단계마다 console.log 추가.
```

## 맑은프레임워크 디버깅

### JSP 디버깅

```jsp
<%
// 변수 출력
System.out.println("boardId: " + boardId);
System.out.println("userId: " + userId);

// DataSet 내용 확인
while(list.next()) {
    System.out.println("id: " + list.i("id") + ", title: " + list.s("title"));
}
%>
```

### SQL 로그

```java
// DAO에서 쿼리 로그
String sql = "SELECT * FROM posts WHERE id = ?";
System.out.println("SQL: " + sql);
System.out.println("Params: " + id);
```

## Workers 디버깅

### console.log

```typescript
export default {
    async fetch(request: Request, env: Env): Promise<Response> {
        console.log('Request URL:', request.url);
        console.log('Method:', request.method);

        const data = await request.json();
        console.log('Request body:', JSON.stringify(data));

        const result = await processData(data);
        console.log('Result:', result);

        return Response.json(result);
    }
};
```

### wrangler tail

```bash
# 실시간 로그
npx wrangler tail

# 에러만 필터링
npx wrangler tail --status error

# POST 요청만
npx wrangler tail --method POST
```

## 브라우저 디버깅

### DevTools 활용

**Console:**
- 에러 메시지 확인
- 변수 값 확인

**Network:**
- API 요청/응답 확인
- 응답 시간 측정

**Sources:**
- 중단점 설정
- 단계별 실행

## API 디버깅

### curl 테스트

```bash
# GET 요청
curl https://api.example.com/posts

# POST 요청
curl -X POST https://api.example.com/posts \
  -H "Content-Type: application/json" \
  -d '{"title":"Test","content":"Content"}'

# 인증 헤더
curl https://api.example.com/protected \
  -H "Authorization: Bearer token"
```

### Postman 사용

1. 요청 생성
2. 헤더 설정
3. Body 설정
4. 응답 확인

## 데이터베이스 디버깅

### EXPLAIN 사용

```sql
-- 쿼리 실행 계획 확인
EXPLAIN SELECT * FROM posts WHERE user_id = 1;

-- 인덱스 사용 여부 확인
EXPLAIN QUERY PLAN SELECT * FROM posts WHERE user_id = 1;
```

### 느린 쿼리 찾기

```
D1 데이터베이스의 느린 쿼리를 찾아줘.

확인:
1. 인덱스 없는 WHERE 절
2. SELECT * 사용
3. LIMIT 없는 쿼리
```

## 실전 프롬프트 예시

### 버그 재현

```
다음 버그를 재현해줘:

증상: 게시글 작성 후 목록에 표시 안 됨

재현 단계:
1. /board/board_write.jsp 접속
2. 제목, 내용 입력
3. 저장 버튼 클릭
4. 목록 페이지로 이동
5. 새 게시글 없음

각 단계마다 로그 추가하고 문제 찾기.
```

### 에러 추적

```
다음 에러를 추적해줘:

TypeError: Cannot read property 'title' of undefined

파일: src/logic/blog/detail.js
라인: 15

확인:
1. 변수가 정의되었는지
2. API 응답이 올바른지
3. null 체크 누락 여부
```

### 성능 병목 찾기

```
/api/posts 엔드포인트가 느려.

디버깅:
1. 각 구간 시간 측정 (console.time)
2. DB 쿼리 분석 (EXPLAIN)
3. 캐싱 여부 확인
4. N+1 문제 확인

병목 지점 찾고 최적화 제안.
```

## 디버깅 체크리스트

- [ ] 에러 메시지를 정확히 읽었는가?
- [ ] 로그를 추가했는가?
- [ ] 변수 값을 확인했는가?
- [ ] API 요청/응답을 확인했는가?
- [ ] 데이터베이스 쿼리를 확인했는가?
- [ ] 브라우저 DevTools를 사용했는가?

## 디버깅 팁

1. **단순화하기**
   - 복잡한 코드를 단순하게 만들기
   - 한 번에 하나씩 테스트

2. **가정 검증하기**
   - "당연히 되겠지" → 확인 필요
   - 모든 변수 값 출력

3. **최근 변경 확인**
   - git diff로 최근 변경 사항 확인
   - 변경 전후 비교

4. **재현 가능하게**
   - 버그 재현 단계 정리
   - 최소 재현 코드 작성

## 관련 문서

- [일반적인 오류 해결](troubleshooting.md)
- [성능 최적화](performance.md)
