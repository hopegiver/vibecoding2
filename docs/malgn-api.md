# 맑은프레임워크 - API 개발 및 연동

맑은프레임워크에서 RESTful API와 AJAX 처리 방법을 학습합니다.

## API 기본 구조

### JSON 응답 API

**경로:** `public_html/api/board_list.jsp`

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

BoardDao board = new BoardDao();
DataSet list = board.findAll("ORDER BY id DESC LIMIT 10");

// 날짜 포맷
while(list.next()) {
    list.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", list.s("reg_date")));
}

j.put("success", true);
j.put("data", list);
j.put("total", list.size());
j.print();

%>
```

**응답 예시:**

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "게시글 제목",
      "content": "내용",
      "reg_date": "20260129120000",
      "reg_date_format": "2026-01-29 12:00"
    }
  ],
  "total": 10
}
```

### POST 요청 처리

**경로:** `public_html/api/board_create.jsp`

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 인증 체크
if(!isLogin) {
    j.put("success", false);
    j.put("message", "로그인이 필요합니다.");
    j.print();
    return;
}

// POST만 허용
if(!m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

// 파라미터 받기
String title = f.get("title");
String content = f.get("content");

// 유효성 검사
if(title.isEmpty()) {
    j.put("success", false);
    j.put("message", "제목을 입력하세요.");
    j.print();
    return;
}

if(content.isEmpty()) {
    j.put("success", false);
    j.put("message", "내용을 입력하세요.");
    j.print();
    return;
}

// DB 저장
BoardDao board = new BoardDao();
DataSet newItem = board.create();
newItem.put("user_id", userId);
newItem.put("title", title);
newItem.put("content", content);
newItem.put("reg_date", m.time("yyyyMMddHHmmss"));
board.save(newItem);

j.put("success", true);
j.put("message", "저장되었습니다.");
j.put("id", newItem.i("id"));
j.print();

%>
```

**AJAX 호출:**

```javascript
fetch('/api/board_create.jsp', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    },
    body: new URLSearchParams({
        title: '게시글 제목',
        content: '게시글 내용'
    })
})
.then(res => res.json())
.then(data => {
    if(data.success) {
        alert(data.message);
        location.href = '/board/board_view.jsp?id=' + data.id;
    } else {
        alert(data.message);
    }
});
```

## CRUD API 패턴

### Create (생성)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin || !m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

String title = f.get("title");
if(title.isEmpty()) {
    j.put("success", false);
    j.put("message", "제목을 입력하세요.");
    j.print();
    return;
}

BoardDao board = new BoardDao();
DataSet newItem = board.create();
newItem.put("user_id", userId);
newItem.put("title", title);
newItem.put("content", f.get("content"));
newItem.put("reg_date", m.time("yyyyMMddHHmmss"));
board.save(newItem);

j.put("success", true);
j.put("id", newItem.i("id"));
j.print();

%>
```

### Read (조회)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

int boardId = m.ri("id");
if(boardId == 0) {
    j.put("success", false);
    j.put("message", "ID를 입력하세요.");
    j.print();
    return;
}

BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    j.put("success", false);
    j.put("message", "게시글을 찾을 수 없습니다.");
    j.print();
    return;
}

info.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", info.s("reg_date")));

j.put("success", true);
j.put("data", info);
j.print();

%>
```

### Update (수정)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin || !m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

int boardId = m.ri("id");
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    j.put("success", false);
    j.put("message", "게시글을 찾을 수 없습니다.");
    j.print();
    return;
}

// 권한 체크
if(info.i("user_id") != userId) {
    j.put("success", false);
    j.put("message", "수정 권한이 없습니다.");
    j.print();
    return;
}

// 수정
info.put("title", f.get("title"));
info.put("content", f.get("content"));
info.put("mod_date", m.time("yyyyMMddHHmmss"));
board.save(info);

j.put("success", true);
j.put("message", "수정되었습니다.");
j.print();

%>
```

### Delete (삭제)

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin || !m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

int boardId = m.ri("id");
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    j.put("success", false);
    j.put("message", "게시글을 찾을 수 없습니다.");
    j.print();
    return;
}

// 권한 체크
if(info.i("user_id") != userId) {
    j.put("success", false);
    j.put("message", "삭제 권한이 없습니다.");
    j.print();
    return;
}

// 삭제
board.delete(boardId);

j.put("success", true);
j.put("message", "삭제되었습니다.");
j.print();

%>
```

## DAO 클래스 작성

### 기본 DAO

**src/dao/BoardDao.java:**

```java
package dao;

import malgnsoft.db.*;

public class BoardDao extends DataObject {
    public BoardDao() {
        this.table = "tb_board";
        this.PK = "id";
    }

    public DataSet findById(int id) {
        return this.find("id = ?", new Object[]{id});
    }

    public DataSet findAll(String orderBy) {
        return this.find("1=1 " + orderBy);
    }

    public DataSet findByUserId(int userId, String orderBy) {
        return this.find("user_id = ? " + orderBy, new Object[]{userId});
    }

    public boolean increaseViewCount(int id) {
        return this.executeUpdate(
            "UPDATE tb_board SET view_count = view_count + 1 WHERE id = ?",
            new Object[]{id}
        );
    }
}
```

### JOIN이 필요한 DAO

```java
package dao;

import malgnsoft.db.*;

public class BoardDao extends DataObject {
    public BoardDao() {
        this.table = "tb_board";
        this.PK = "id";
    }

    public DataSet findWithUser(int id) {
        String sql = "SELECT b.*, u.name as user_name, u.email as user_email " +
                     "FROM tb_board b " +
                     "LEFT JOIN tb_user u ON b.user_id = u.id " +
                     "WHERE b.id = ?";
        return this.executeQuery(sql, new Object[]{id});
    }

    public DataSet findAllWithUser(String orderBy) {
        String sql = "SELECT b.*, u.name as user_name " +
                     "FROM tb_board b " +
                     "LEFT JOIN tb_user u ON b.user_id = u.id " +
                     orderBy;
        return this.executeQuery(sql);
    }
}
```

### 검색 기능이 있는 DAO

```java
public class BoardDao extends DataObject {
    // ...

    public DataSet search(String keyword, String orderBy) {
        String sql = "SELECT b.*, u.name as user_name " +
                     "FROM tb_board b " +
                     "LEFT JOIN tb_user u ON b.user_id = u.id " +
                     "WHERE b.title LIKE ? OR b.content LIKE ? " +
                     orderBy;
        String keywordPattern = "%" + keyword + "%";
        return this.executeQuery(sql, new Object[]{keywordPattern, keywordPattern});
    }
}
```

## AJAX 폼 제출

### 기본 패턴

**HTML:**

```html
<form id="boardForm">
    <input type="text" name="title" class="form-control" placeholder="제목" required>
    <textarea name="content" class="form-control" placeholder="내용" required></textarea>
    <button type="submit" class="btn btn-primary">저장</button>
</form>

<script>
document.getElementById('boardForm').addEventListener('submit', function(e) {
    e.preventDefault();

    const formData = new FormData(this);

    fetch('/api/board_create.jsp', {
        method: 'POST',
        body: formData
    })
    .then(res => res.json())
    .then(data => {
        alert(data.message);
        if(data.success) {
            location.href = '/board/board_view.jsp?id=' + data.id;
        }
    })
    .catch(err => {
        alert('오류가 발생했습니다.');
        console.error(err);
    });
});
</script>
```

### jQuery 사용

```html
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
$('#boardForm').on('submit', function(e) {
    e.preventDefault();

    $.ajax({
        url: '/api/board_create.jsp',
        type: 'POST',
        data: $(this).serialize(),
        dataType: 'json',
        success: function(data) {
            alert(data.message);
            if(data.success) {
                location.href = '/board/board_view.jsp?id=' + data.id;
            }
        },
        error: function() {
            alert('오류가 발생했습니다.');
        }
    });
});
</script>
```

## 파일 업로드 API

### 파일 업로드 처리

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin || !m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

// 파일 업로드 처리
Upload upload = new Upload(request);
upload.setUploadDir("/upload/board");  // 업로드 경로
upload.setMaxFileSize(10 * 1024 * 1024);  // 10MB

if(!upload.parseRequest()) {
    j.put("success", false);
    j.put("message", "파일 업로드 실패");
    j.print();
    return;
}

String filename = upload.getFileName("file");
if(filename == null) {
    j.put("success", false);
    j.put("message", "파일을 선택하세요.");
    j.print();
    return;
}

String savedPath = upload.getSavedFileName("file");

j.put("success", true);
j.put("filename", filename);
j.put("path", savedPath);
j.print();

%>
```

**HTML:**

```html
<form id="uploadForm">
    <input type="file" name="file" required>
    <button type="submit">업로드</button>
</form>

<script>
document.getElementById('uploadForm').addEventListener('submit', function(e) {
    e.preventDefault();

    const formData = new FormData(this);

    fetch('/api/file_upload.jsp', {
        method: 'POST',
        body: formData
    })
    .then(res => res.json())
    .then(data => {
        if(data.success) {
            alert('업로드 완료: ' + data.filename);
        } else {
            alert(data.message);
        }
    });
});
</script>
```

## 트랜잭션 처리

### 여러 테이블 동시 저장

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin || !m.isPost()) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}

DB db = new DB();

boolean success = db.transaction(new Runnable() {
    public void run() {
        // 게시글 저장
        BoardDao board = new BoardDao();
        DataSet newBoard = board.create();
        newBoard.put("user_id", userId);
        newBoard.put("title", f.get("title"));
        newBoard.put("content", f.get("content"));
        newBoard.put("reg_date", m.time("yyyyMMddHHmmss"));
        board.save(newBoard);

        int boardId = newBoard.i("id");

        // 태그 저장
        String[] tags = f.getArray("tags");
        TagDao tag = new TagDao();
        for(String tagName : tags) {
            DataSet newTag = tag.create();
            newTag.put("board_id", boardId);
            newTag.put("tag_name", tagName);
            tag.save(newTag);
        }
    }
});

if(success) {
    j.put("success", true);
    j.put("message", "저장되었습니다.");
} else {
    j.put("success", false);
    j.put("message", "저장 실패");
}
j.print();

%>
```

## 실전 프롬프트 예시

### 좋아요 API 추가

```
게시글 좋아요 API를 만들어줘.

경로: /api/board_like.jsp
메서드: POST
파라미터: board_id

기능:
- 로그인 체크
- 중복 좋아요 방지 (tb_board_like 테이블)
- tb_board.like_count 증가
- JSON 응답 (success, message, like_count)

DAO도 같이 만들어줘.
```

### 댓글 CRUD API 추가

```
댓글 CRUD API를 만들어줘.

테이블: tb_comment (board_id, user_id, content, reg_date)

API 목록:
- /api/comment_list.jsp (GET) - board_id로 조회
- /api/comment_create.jsp (POST) - 댓글 작성
- /api/comment_delete.jsp (POST) - 댓글 삭제 (본인만)

CommentDao도 같이 작성.
```

### 검색 API 추가

```
통합 검색 API를 만들어줘.

경로: /api/search.jsp
파라미터: keyword, type (board|user|product)

응답:
- 게시글, 회원, 상품 통합 검색
- 각 타입별 최대 5개씩
- 제목/내용/이름 등에서 검색

JSON 응답에 type별로 그룹화.
```

## 체크리스트

API 개발 시 확인사항:

- [ ] contentType을 application/json으로 설정했는가?
- [ ] 인증이 필요한 API에서 isLogin 체크했는가?
- [ ] POST 요청만 허용해야 하는 경우 m.isPost() 체크했는가?
- [ ] 파라미터 유효성 검사를 했는가?
- [ ] 권한 체크를 했는가? (수정/삭제 시)
- [ ] 적절한 에러 메시지를 반환하는가?
- [ ] 성공/실패 여부를 명확히 반환하는가?
- [ ] SQL Injection 방지를 위해 PreparedStatement 패턴을 사용했는가?

## 자주 하는 실수

### 1. contentType 누락

```jsp
<!-- ❌ 잘못된 코드 -->
<%@ page %><%@ include file="/init.jsp" %>

<!-- ✅ 올바른 코드 -->
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %>
```

### 2. 에러 후 return 누락

```jsp
// ❌ 잘못된 코드
if(!isLogin) {
    j.put("success", false);
    j.print();
    // return 누락!
}
// 이후 코드 계속 실행됨

// ✅ 올바른 코드
if(!isLogin) {
    j.put("success", false);
    j.print();
    return;
}
```

### 3. AJAX 오류 처리 누락

```javascript
// ❌ 잘못된 코드
fetch('/api/board_create.jsp', {
    method: 'POST',
    body: formData
})
.then(res => res.json())
.then(data => {
    alert(data.message);
});
// 네트워크 오류 처리 없음

// ✅ 올바른 코드
fetch('/api/board_create.jsp', {
    method: 'POST',
    body: formData
})
.then(res => res.json())
.then(data => {
    alert(data.message);
})
.catch(err => {
    alert('오류가 발생했습니다.');
    console.error(err);
});
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [데이터베이스 작업](malgn-database.md)
- [보안 가이드라인](security.md)
- [코딩 규칙](coding-rules.md)
