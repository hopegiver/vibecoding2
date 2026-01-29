# 맑은프레임워크 - 데이터베이스 작업

맑은프레임워크의 DataSet, ListManager, DAO 클래스를 활용한 데이터베이스 작업 방법을 학습합니다.

## DataSet 기본

### DataSet이란?

`DataSet`은 맑은프레임워크의 핵심 데이터 구조로, 데이터베이스 쿼리 결과를 담는 컬렉션입니다.

**특징:**
- 여러 행(row)을 담을 수 있음
- `next()` 메서드로 행 이동
- `put()`, `get()` 메서드로 데이터 조작
- Null 안전 (빈 문자열 반환)

### 기본 사용법

```jsp
BoardDao board = new BoardDao();
DataSet list = board.findAll("ORDER BY id DESC");

// 행 순회
while(list.next()) {
    int id = list.i("id");
    String title = list.s("title");
    String regDate = list.s("reg_date");

    // 데이터 추가/수정
    list.put("reg_date_format", m.time("yyyy-MM-dd", regDate));
}

// 크기
int size = list.size();

// 처음으로 되돌리기
list.first();
```

### DataSet 메서드

```jsp
// 문자열 가져오기
String title = list.s("title");           // 없으면 빈 문자열
String title = list.s("title", "기본값");  // 기본값 지정

// 숫자 가져오기
int id = list.i("id");                    // 없으면 0
int id = list.i("id", 1);                 // 기본값 지정

// 값 설정
list.put("title", "새 제목");
list.put("view_count", 100);

// 행 이동
boolean hasNext = list.next();            // 다음 행으로 (있으면 true)
list.first();                             // 첫 행으로
list.last();                              // 마지막 행으로

// 크기
int size = list.size();
boolean isEmpty = list.isEmpty();
```

## DAO 클래스 패턴

### 기본 DAO 구조

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

**필수 설정:**
- `table`: 테이블 이름
- `PK`: Primary Key 컬럼명

### CRUD 메서드

```java
public class BoardDao extends DataObject {
    public BoardDao() {
        this.table = "tb_board";
        this.PK = "id";
    }

    // Create
    public DataSet createNew(int userId, String title, String content) {
        DataSet item = this.create();
        item.put("user_id", userId);
        item.put("title", title);
        item.put("content", content);
        item.put("reg_date", Malgn.time("yyyyMMddHHmmss"));
        this.save(item);
        return item;
    }

    // Read - 단일
    public DataSet findById(int id) {
        return this.find("id = ?", new Object[]{id});
    }

    // Read - 목록
    public DataSet findAll(String orderBy) {
        return this.find("1=1 " + orderBy);
    }

    // Read - 조건
    public DataSet findByUserId(int userId, String orderBy) {
        return this.find("user_id = ? " + orderBy, new Object[]{userId});
    }

    // Update (save 메서드 사용)
    // save()는 자동으로 UPDATE/INSERT 판단

    // Delete
    public boolean deleteById(int id) {
        return this.delete(id);
    }

    // Custom 쿼리
    public boolean increaseViewCount(int id) {
        String sql = "UPDATE tb_board SET view_count = view_count + 1 WHERE id = ?";
        return this.executeUpdate(sql, new Object[]{id});
    }
}
```

### JOIN 쿼리

```java
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

    public DataSet findAllWithUser(String where, String orderBy) {
        String sql = "SELECT b.*, u.name as user_name " +
                     "FROM tb_board b " +
                     "LEFT JOIN tb_user u ON b.user_id = u.id " +
                     "WHERE " + where + " " + orderBy;
        return this.executeQuery(sql);
    }
}
```

**JSP에서 사용:**

```jsp
BoardDao board = new BoardDao();
DataSet info = board.findWithUser(boardId);
if(!info.next()) {
    m.jsAlert("게시글을 찾을 수 없습니다.");
    return;
}

// user_name, user_email 사용 가능
String userName = info.s("user_name");
```

## ListManager (페이징)

### 기본 사용법

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(10);                    // 페이지당 10개
lm.setTable("tb_board");
lm.setFields("*");
lm.setOrderBy("id DESC");

DataSet list = lm.getDataSet();

// 템플릿에 전달
p.setLoop("list", list);
p.setVar("total", lm.getTotalNum());
p.setVar("pager", lm.getPaging());
```

**HTML:**

```html
<p>전체 {total}건</p>

<table class="table">
{#loop list}
    <tr>
        <td>{list.id}</td>
        <td>{list.title}</td>
    </tr>
{#/loop}
</table>

<nav>
    <ul class="pagination">
        {pager}
    </ul>
</nav>
```

### JOIN과 함께 사용

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(10);
lm.setTable("tb_board b LEFT JOIN tb_user u ON b.user_id = u.id");
lm.setFields("b.*, u.name as user_name");
lm.setOrderBy("b.id DESC");

DataSet list = lm.getDataSet();

// 날짜 포맷 추가
while(list.next()) {
    list.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", list.s("reg_date")));
}

p.setLoop("list", list);
```

### 검색 조건 추가

```jsp
// 폼 필드 설정
f.addElement("keyword", null, null);
f.addElement("category", null, null);

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(10);
lm.setTable("tb_board b LEFT JOIN tb_user u ON b.user_id = u.id");
lm.setFields("b.*, u.name as user_name");

// 검색 조건
String keyword = f.get("keyword");
if(!keyword.isEmpty()) {
    lm.addSearch("b.title,b.content", keyword, "LIKE");
}

String category = f.get("category");
if(!category.isEmpty()) {
    lm.addWhere("b.category = '" + category + "'");
}

lm.setOrderBy("b.id DESC");

DataSet list = lm.getDataSet();
```

**addSearch 메서드:**
- `LIKE`: 부분 일치 (`%keyword%`)
- `=`: 완전 일치
- `!=`: 불일치

### 정렬 옵션

```jsp
// 정렬 필드를 파라미터로 받기
String orderBy = m.rs("order_by", "id");  // 기본값: id
String orderDir = m.rs("order_dir", "DESC");  // 기본값: DESC

// 허용된 필드만 사용 (보안)
if(!orderBy.matches("^(id|title|reg_date|view_count)$")) {
    orderBy = "id";
}
if(!orderDir.matches("^(ASC|DESC)$")) {
    orderDir = "DESC";
}

lm.setOrderBy(orderBy + " " + orderDir);
```

## CRUD 패턴

### Create (생성)

```jsp
BoardDao board = new BoardDao();

// 새 DataSet 생성
DataSet newItem = board.create();
newItem.put("user_id", userId);
newItem.put("title", f.get("title"));
newItem.put("content", f.get("content"));
newItem.put("view_count", 0);
newItem.put("reg_date", m.time("yyyyMMddHHmmss"));

// 저장 (INSERT)
board.save(newItem);

// 생성된 ID 가져오기
int newId = newItem.i("id");
```

### Read (조회)

```jsp
BoardDao board = new BoardDao();

// 단일 조회
DataSet info = board.findById(boardId);
if(!info.next()) {
    m.jsAlert("게시글을 찾을 수 없습니다.");
    return;
}

String title = info.s("title");
String content = info.s("content");

// 목록 조회
DataSet list = board.findAll("ORDER BY id DESC LIMIT 10");
while(list.next()) {
    // 처리
}
```

### Update (수정)

```jsp
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    m.jsAlert("게시글을 찾을 수 없습니다.");
    return;
}

// 수정
info.put("title", f.get("title"));
info.put("content", f.get("content"));
info.put("mod_date", m.time("yyyyMMddHHmmss"));

// 저장 (UPDATE)
board.save(info);
```

**주의:** `save()` 메서드는 자동으로 INSERT/UPDATE를 판단합니다.
- PK가 있으면: UPDATE
- PK가 없으면: INSERT

### Delete (삭제)

```jsp
BoardDao board = new BoardDao();

// ID로 삭제
boolean success = board.delete(boardId);

if(success) {
    m.jsAlert("삭제되었습니다.");
    m.jsReplace("/board/board_list.jsp");
} else {
    m.jsAlert("삭제 실패");
}
```

## 트랜잭션

### 기본 트랜잭션

```jsp
DB db = new DB();

boolean success = db.transaction(new Runnable() {
    public void run() {
        // 게시글 저장
        BoardDao board = new BoardDao();
        DataSet newBoard = board.create();
        newBoard.put("title", "제목");
        board.save(newBoard);

        int boardId = newBoard.i("id");

        // 첨부파일 저장
        AttachDao attach = new AttachDao();
        DataSet newAttach = attach.create();
        newAttach.put("board_id", boardId);
        newAttach.put("filename", "file.jpg");
        attach.save(newAttach);
    }
});

if(success) {
    j.put("success", true);
} else {
    j.put("success", false);
    j.put("message", "저장 실패");
}
```

**동작:**
- 모든 작업 성공 시 COMMIT
- 하나라도 실패 시 ROLLBACK

## 날짜 처리

### 날짜 저장

맑은프레임워크는 날짜를 **VARCHAR(14)** 형식으로 저장합니다.

```jsp
// 현재 시간
String now = m.time("yyyyMMddHHmmss");
// 예: "20260129153045"

// DataSet에 저장
info.put("reg_date", now);
info.put("mod_date", now);
```

**형식:** YYYYMMDDHHmmss (14자리)

### 날짜 포맷팅

```jsp
DataSet list = board.findAll("ORDER BY id DESC");

while(list.next()) {
    String regDate = list.s("reg_date");

    // 포맷 변환
    list.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", regDate));
    list.put("reg_date_short", m.time("yyyy-MM-dd", regDate));
}

p.setLoop("list", list);
```

**HTML:**

```html
{#loop list}
    <p>작성일: {list.reg_date_format}</p>
{#/loop}
```

### 날짜 비교

```jsp
String now = m.time("yyyyMMddHHmmss");
String expireDate = info.s("expire_date");

if(now.compareTo(expireDate) > 0) {
    // 만료됨
}
```

## 실전 프롬프트 예시

### 게시판 CRUD 추가

```
게시판 CRUD 기능을 만들어줘.

테이블: tb_board
필드: id, user_id, title, content, view_count, reg_date

DAO: BoardDao
페이지:
- board_list.jsp (목록, 페이징 10개)
- board_view.jsp (상세보기, 조회수 증가)
- board_write.jsp (작성, Postback 패턴)
- board_modify.jsp (수정, 권한 체크)
- board_delete.jsp (삭제, POST만 허용)

ListManager 사용, JOIN으로 작성자 이름 표시.
```

### 검색 기능 추가

```
게시판 검색 기능을 추가해줬으면 해.

검색 조건:
- keyword: 제목+내용 (LIKE)
- category: 카테고리 (완전 일치)
- date_from, date_to: 작성일 범위

ListManager.addSearch, addWhere 사용.
검색 폼도 같이 만들어줘.
```

### 댓글 기능 추가

```
게시글에 댓글 기능을 추가해줘.

테이블: tb_comment
필드: id, board_id, user_id, content, reg_date

DAO: CommentDao
API: /api/comment_list.jsp, /api/comment_create.jsp, /api/comment_delete.jsp

board_view.jsp에 댓글 목록과 작성 폼 추가.
AJAX로 처리.
```

## 체크리스트

데이터베이스 작업 시 확인사항:

- [ ] DAO 클래스에서 table과 PK를 설정했는가?
- [ ] DataSet.next() 호출 후 데이터를 사용하는가?
- [ ] SQL Injection 방지를 위해 PreparedStatement 패턴을 사용하는가?
- [ ] 날짜 필드를 VARCHAR(14)로 저장하는가?
- [ ] 날짜 포맷팅을 했는가?
- [ ] ListManager에서 JOIN 사용 시 테이블 별칭을 붙였는가?
- [ ] 검색 조건을 안전하게 처리했는가?
- [ ] 트랜잭션이 필요한 경우 db.transaction()을 사용했는가?

## 자주 하는 실수

### 1. next() 호출 누락

```jsp
// ❌ 잘못된 코드
DataSet info = board.findById(id);
String title = info.s("title");  // 데이터 없음!

// ✅ 올바른 코드
DataSet info = board.findById(id);
if(!info.next()) {
    m.jsAlert("데이터를 찾을 수 없습니다.");
    return;
}
String title = info.s("title");
```

### 2. 날짜 형식 오류

```jsp
// ❌ 잘못된 코드
info.put("reg_date", new Date());  // Java Date 객체

// ✅ 올바른 코드
info.put("reg_date", m.time("yyyyMMddHHmmss"));
```

### 3. SQL Injection 위험

```jsp
// ❌ 잘못된 코드 (위험!)
String keyword = f.get("keyword");
DataSet list = board.find("title LIKE '%" + keyword + "%'");

// ✅ 올바른 코드
String keyword = f.get("keyword");
lm.addSearch("title", keyword, "LIKE");
```

### 4. JOIN 별칭 누락

```jsp
// ❌ 잘못된 코드
lm.setTable("tb_board LEFT JOIN tb_user ON tb_board.user_id = tb_user.id");
lm.setFields("*");  // 컬럼명 충돌!

// ✅ 올바른 코드
lm.setTable("tb_board b LEFT JOIN tb_user u ON b.user_id = u.id");
lm.setFields("b.*, u.name as user_name");
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [API 개발 및 연동](malgn-api.md)
- [보안 가이드라인](security.md)
- [코딩 규칙](coding-rules.md)
