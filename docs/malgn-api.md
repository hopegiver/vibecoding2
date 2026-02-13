# 맑은프레임워크 - API 개발 및 연동

## API 아키텍처

맑은프레임워크는 **중앙 라우터 방식**의 API 구조를 사용한다.

```
WEB-INF/web.xml         → /api/* 요청을 /api/index.jsp로 매핑
public_html/api/init.jsp → JWT 인증, CORS, 퍼블릭 라우트 설정
public_html/api/index.jsp → API 라우터 (엔드포인트 분기)
```

### web.xml 매핑

```xml
<servlet-mapping>
    <servlet-name>api</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

모든 `/api/*` 요청은 `index.jsp`를 거쳐 라우팅된다.

### api/init.jsp - API 초기화

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ page import="java.util.*,java.io.*,malgnsoft.db.*,malgnsoft.util.*,malgnsoft.json.*,dao.*" %><%

Malgn m = new Malgn(request, response, out);
Form f = new Form();
f.setRequest(request);
Json j = new Json(out);
RestAPI api = new RestAPI(request, response);

// CORS 설정
api.cors();

// OPTIONS 요청 처리
if(api.handlePreflight()) return;

// 퍼블릭 라우팅 (인증 불필요)
api.publicRoute(
    "/api/auth/*",
    "/api/board/list",
    "/api/board/view/*"
);

// JWT 인증
if(!api.auth()) return;

// 인증된 사용자 정보
int userId = api.getDataInt("user_id");
String userName = api.getData("user_name");
boolean isLogin = (userId > 0);

%>
```

핵심 포인트:
- `api.publicRoute()`로 인증 없이 접근 가능한 경로를 지정
- `api.auth()`가 JWT 토큰 검증을 처리하고, 실패 시 자동으로 401 응답
- 인증 후 `api.getDataInt()`, `api.getData()`로 사용자 정보 접근

## RestAPI 라우팅

### HTTP 메서드별 라우팅

`api.get()`, `api.post()`, `api.put()`, `api.delete()`로 메서드와 경로 패턴을 매칭한다.

```jsp
<%@ include file="init.jsp" %><%

// 목록 조회: GET /api/board
api.get("/", () -> {
    BoardDao board = new BoardDao();
    DataSet list = board.findAll("ORDER BY id DESC LIMIT 10");
    j.success("조회 성공");
    j.put("data", list);
    j.print();
});

// 상세 조회: GET /api/board/:id
api.get("/:id", () -> {
    int id = api.paramInt("id");
    BoardDao board = new BoardDao();
    DataSet info = board.find("id = " + id);
    if(!info.next()) { j.error("게시글을 찾을 수 없습니다."); return; }
    j.success("조회 성공");
    j.put("data", info);
    j.print();
});

// 생성: POST /api/board
api.post("/", () -> {
    BoardDao board = new BoardDao();
    board.item("user_id", userId);
    board.item("title", f.get("title"));
    board.item("content", f.get("content"));
    board.item("reg_date", m.time("yyyyMMddHHmmss"));
    board.insert();

    j.success("저장되었습니다.");
    j.print();
});

// 수정: PUT /api/board/:id
api.put("/:id", () -> {
    int id = api.paramInt("id");
    BoardDao board = new BoardDao();
    board.item("title", f.get("title"));
    board.item("content", f.get("content"));
    board.update("id = " + id);

    j.success("수정되었습니다.");
    j.print();
});

// 삭제: DELETE /api/board/:id
api.delete("/:id", () -> {
    int id = api.paramInt("id");
    BoardDao board = new BoardDao();
    board.delete("id = " + id);

    j.success("삭제되었습니다.");
    j.print();
});

%>
```

### JSON 응답 패턴

```jsp
// 성공 응답
j.success("저장되었습니다.");
j.put("data", list);
j.print();

// 에러 응답 (자동으로 success: false 설정)
j.error("게시글을 찾을 수 없습니다.");
// j.error() 호출 후에는 return 필요
```

## DAO 패턴

### 기본 DAO 클래스

**src/dao/BoardDao.java:**

```java
package dao;

import malgnsoft.db.*;

public class BoardDao extends DataObject {
    public BoardDao() {
        this.table = "tb_board";
        this.PK = "id";
    }
}
```

### DAO CRUD 사용법

```jsp
BoardDao board = new BoardDao();

// INSERT
board.item("title", f.get("title"));
board.item("user_id", userId);
board.insert();

// UPDATE
board.item("title", f.get("title"));
board.update("id = " + id);

// DELETE
board.delete("id = " + id);

// SELECT
DataSet list = board.find("user_id = " + userId + " ORDER BY id DESC");
while(list.next()) {
    list.s("title");  // String 값
    list.i("id");     // int 값
}

// 단건 조회
DataSet info = board.find("id = " + id);
if(!info.next()) { /* 없음 */ }
```

### JOIN 쿼리

```java
public DataSet findWithUser(int id) {
    String sql = "SELECT b.*, u.name as user_name " +
                 "FROM tb_board b " +
                 "LEFT JOIN tb_user u ON b.user_id = u.id " +
                 "WHERE b.id = ?";
    return this.executeQuery(sql, new Object[]{id});
}
```

## AJAX 폼 처리

맑은프레임워크는 `data-ajax="true"` 속성으로 폼의 AJAX 제출을 자동 처리한다. `common.js`가 이를 인터셉트하여 처리한다.

### data-ajax 폼

```html
<!-- 저장 후 리다이렉트 -->
<form action="/api/board" method="post" data-ajax="true" data-redirect="/board/list.jsp">
    <input type="text" name="title" required>
    <textarea name="content"></textarea>
    <button type="submit">저장</button>
</form>

<!-- 저장 후 메시지만 표시 -->
<form action="/api/board" method="post" data-ajax="true" data-success="저장되었습니다.">
    <input type="text" name="title" required>
    <button type="submit">저장</button>
</form>
```

- `data-ajax="true"`: AJAX로 폼 제출 (common.js가 자동 처리)
- `data-redirect`: 성공 시 이동할 URL
- `data-success`: 성공 시 표시할 메시지

### 수동 fetch 호출

자동 폼 처리가 아닌 직접 호출이 필요한 경우:

```javascript
fetch('/api/board', {
    method: 'POST',
    body: new FormData(document.getElementById('myForm'))
})
.then(res => res.json())
.then(data => {
    if(data.success) {
        alert(data.message);
    } else {
        alert(data.message);
    }
})
.catch(err => console.error(err));
```

## 파일 업로드 API

```jsp
api.post("/upload", () -> {
    Upload upload = new Upload(request);
    upload.setUploadDir("/upload/board");
    upload.setMaxFileSize(10 * 1024 * 1024);  // 10MB

    if(!upload.parseRequest()) {
        j.error("파일 업로드 실패");
        return;
    }

    String filename = upload.getFileName("file");
    String savedPath = upload.getSavedFileName("file");

    j.success("업로드 완료");
    j.put("filename", filename);
    j.put("path", savedPath);
    j.print();
});
```

## 자주 하는 실수

**1. contentType 누락**
```jsp
<!-- 잘못됨: contentType 없음 -->
<%@ page %><%@ include file="init.jsp" %>

<!-- 올바름 -->
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="init.jsp" %>
```

**2. j.error() 후 return 누락**
```jsp
// 잘못됨: return 없이 이후 코드가 계속 실행됨
if(!info.next()) {
    j.error("데이터 없음");
}
// ...이후 코드 실행됨

// 올바름
if(!info.next()) {
    j.error("데이터 없음");
    return;
}
```

**3. DAO에서 create()/save() 사용 (잘못된 패턴)**
```jsp
// 잘못됨
DataSet newItem = board.create();
newItem.put("title", title);
board.save(newItem);

// 올바름
board.item("title", title);
board.insert();
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [데이터베이스 작업](malgn-database.md)
- [보안 가이드라인](security.md)
- [코딩 규칙](coding-rules.md)
