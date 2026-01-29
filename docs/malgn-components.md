# 맑은프레임워크 - 컴포넌트 개발

맑은프레임워크의 템플릿 엔진을 활용한 컴포넌트 개발 방법을 학습합니다.

## Page 클래스 기본

### 주요 메서드

```jsp
Page p = new Page();
p.setRequest(request);
p.setPageContext(pageContext);
p.setWriter(out);

// 레이아웃 설정
p.setLayout("main");

// 본문 템플릿 설정
p.setBody("board.board_list");

// 변수 전달
p.setVar("title", "게시판");
p.setVar("total", 100);

// 반복 데이터 전달
p.setLoop("list", dataSet);

// 렌더링
p.display();
```

## 변수 전달 (setVar)

### 단순 값

```jsp
// 문자열
p.setVar("userName", "홍길동");
p.setVar("message", "환영합니다!");

// 숫자
p.setVar("total", 100);
p.setVar("userId", 123);

// 불린
p.setVar("isLogin", true);
p.setVar("isAdmin", false);
```

**HTML:**

```html
<p>사용자: {userName}</p>
<p>메시지: {message}</p>
<p>총 {total}건</p>
```

### DataSet 객체 전달

```jsp
UserDao user = new UserDao();
DataSet info = user.findById(userId);
info.next();  // ⭐ 첫 행으로 이동 필수

p.setVar("info", info);
```

**HTML:**

```html
<dl>
    <dt>이름</dt>
    <dd>{info.name}</dd>

    <dt>이메일</dt>
    <dd>{info.email}</dd>

    <dt>가입일</dt>
    <dd>{info.reg_date}</dd>
</dl>
```

### 날짜 포맷팅

```jsp
DataSet info = user.findById(userId);
info.next();

// 날짜 포맷 추가
info.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", info.s("reg_date")));

p.setVar("info", info);
```

**HTML:**

```html
<p>가입일: {info.reg_date_format}</p>
```

## 반복 처리 (setLoop)

### 기본 반복

```jsp
BoardDao board = new BoardDao();
DataSet list = board.findAll("ORDER BY id DESC LIMIT 10");

// 날짜 포맷 추가
while(list.next()) {
    list.put("reg_date_format", m.time("yyyy-MM-dd", list.s("reg_date")));
}

p.setLoop("list", list);
```

**HTML:**

```html
<ul>
{#loop list}
    <li>
        <a href="board_view.jsp?id={list.id}">{list.title}</a>
        <span class="text-muted">({list.reg_date_format})</span>
    </li>
{#/loop}
</ul>
```

**주의:** `{#loop list}`에서 `list`는 setLoop의 첫 번째 인자와 일치해야 함

### 테이블 반복

```jsp
p.setLoop("users", userList);
```

**HTML:**

```html
<table class="table">
    <thead>
        <tr>
            <th>ID</th>
            <th>이름</th>
            <th>이메일</th>
            <th>가입일</th>
        </tr>
    </thead>
    <tbody>
    {#loop users}
        <tr>
            <td>{users.id}</td>
            <td>{users.name}</td>
            <td>{users.email}</td>
            <td>{users.reg_date_format}</td>
        </tr>
    {#/loop}
    </tbody>
</table>
```

### 빈 목록 처리

```html
{#loop list}
    <div class="card">
        <h5>{list.title}</h5>
        <p>{list.content}</p>
    </div>
{#empty}
    <div class="alert alert-info">
        등록된 게시글이 없습니다.
    </div>
{#/loop}
```

**동작:**
- `list`에 데이터가 있으면: 반복
- `list`가 비어있으면: `{#empty}` 블록 표시

### 반복 인덱스

```html
{#loop list}
    <div class="item">
        <span class="badge">{__LOOP_CNT__}</span>
        <h5>{list.title}</h5>
    </div>
{#/loop}
```

**변수:**
- `{__LOOP_CNT__}`: 1부터 시작하는 인덱스 (1, 2, 3, ...)
- `{__LOOP_IDX__}`: 0부터 시작하는 인덱스 (0, 1, 2, ...)

## 조건문 (if)

### 기본 조건문

```jsp
p.setVar("isLogin", true);
p.setVar("userName", "홍길동");
```

**HTML:**

```html
{#if isLogin}
    <p>안녕하세요, {userName}님!</p>
    <a href="/member/logout.jsp">로그아웃</a>
{#else}
    <a href="/member/login.jsp">로그인</a>
{#/if}
```

### 비교 연산

```jsp
p.setVar("userLevel", 9);
```

**HTML:**

```html
{#if userLevel >= 9}
    <span class="badge bg-danger">관리자</span>
{#else}
    <span class="badge bg-secondary">일반회원</span>
{#/if}
```

**지원 연산자:**
- `==`, `!=`
- `>`, `<`, `>=`, `<=`

### 다중 조건 (else if)

```jsp
p.setVar("status", "pending");
```

**HTML:**

```html
{#if status == "approved"}
    <span class="badge bg-success">승인됨</span>
{#elseif status == "pending"}
    <span class="badge bg-warning">대기중</span>
{#elseif status == "rejected"}
    <span class="badge bg-danger">거부됨</span>
{#else}
    <span class="badge bg-secondary">알 수 없음</span>
{#/if}
```

### 반복 내 조건문

```html
{#loop list}
    <tr class="{#if list.user_id == userId}table-primary{#/if}">
        <td>{list.title}</td>
        <td>{list.user_name}</td>
        <td>
            {#if list.user_id == userId}
                <a href="board_modify.jsp?id={list.id}" class="btn btn-sm btn-primary">수정</a>
                <a href="board_delete.jsp?id={list.id}" class="btn btn-sm btn-danger">삭제</a>
            {#/if}
        </td>
    </tr>
{#/loop}
```

## 폼 스크립트 (Form 클래스)

### 기본 사용법

```jsp
f.addElement("email", null, "required|email");
f.addElement("passwd", null, "required|minLength:6");
f.addElement("name", null, "required");

p.setVar("form_script", f.getScript());
```

**HTML:**

```html
<form id="registerForm" method="post">
    <input type="email" name="email" class="form-control" placeholder="이메일">
    <input type="password" name="passwd" class="form-control" placeholder="비밀번호">
    <input type="text" name="name" class="form-control" placeholder="이름">
    <button type="submit" class="btn btn-primary">가입하기</button>
</form>

{form_script}
```

**생성되는 스크립트:**
- 폼 제출 시 자동 유효성 검사
- 오류 시 alert로 메시지 표시
- 첫 번째 오류 필드에 포커스

### 유효성 규칙

```jsp
// 필수
f.addElement("title", null, "required");

// 이메일
f.addElement("email", null, "required|email");

// 최소/최대 길이
f.addElement("passwd", null, "required|minLength:6|maxLength:20");

// 숫자
f.addElement("age", null, "required|number");

// 정규식
f.addElement("phone", null, "required|match:/^010-\\d{4}-\\d{4}$/");
```

### 기본값 설정

```jsp
// 수정 페이지
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
info.next();

f.addElement("title", info.s("title"), "required");
f.addElement("content", info.s("content"), "required");

p.setVar("info", info);
p.setVar("form_script", f.getScript());
```

**HTML:**

```html
<form method="post">
    <input type="text" name="title" value="{info.title}" class="form-control">
    <textarea name="content" class="form-control">{info.content}</textarea>
    <button type="submit" class="btn btn-primary">수정</button>
</form>

{form_script}
```

## 재사용 가능한 컴포넌트

### 부분 템플릿 (Include)

**JSP에서 여러 템플릿 사용:**

```jsp
p.setLayout("main");
p.setBody("board.board_view");
p.setVar("info", boardInfo);

// 댓글 목록 추가
p.setLoop("comments", commentList);
```

**HTML - board_view.html:**

```html
<div class="container">
    <h1>{info.title}</h1>
    <div class="content">{info.content}</div>

    <hr>

    <h3>댓글</h3>
    {#loop comments}
        <div class="comment">
            <strong>{comments.user_name}</strong>
            <p>{comments.content}</p>
        </div>
    {#/loop}
</div>
```

### 네비게이션 컴포넌트

**JSP - init.jsp에 추가:**

```jsp
// 현재 경로
String currentPath = request.getRequestURI();
p.setVar("currentPath", currentPath);
```

**HTML - layout_main.html:**

```html
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="/">My Site</a>
        <ul class="navbar-nav">
            <li class="nav-item">
                <a class="nav-link {#if currentPath == "/main/index.jsp"}active{#/if}"
                   href="/main/index.jsp">홈</a>
            </li>
            <li class="nav-item">
                <a class="nav-link {#if currentPath == "/board/board_list.jsp"}active{#/if}"
                   href="/board/board_list.jsp">게시판</a>
            </li>
        </ul>
    </div>
</nav>
```

### 페이징 컴포넌트

**JSP:**

```jsp
ListManager lm = new ListManager();
lm.setRequest(request);
lm.setListNum(10);
// ... 설정 ...

p.setVar("pager", lm.getPaging());
```

**HTML:**

```html
<nav>
    <ul class="pagination">
        {pager}
    </ul>
</nav>
```

**생성되는 HTML:**
```html
<li class="page-item"><a class="page-link" href="?page=1">1</a></li>
<li class="page-item active"><a class="page-link" href="?page=2">2</a></li>
<li class="page-item"><a class="page-link" href="?page=3">3</a></li>
```

## 실전 프롬프트 예시

### 카드 리스트 컴포넌트

```
상품 목록 페이지를 만들어줘.

경로: /product/product_list.jsp
DAO: ProductDao
레이아웃: 3열 카드 그리드 (Bootstrap)

카드 내용:
- 썸네일 이미지
- 제품명
- 가격
- 상세보기 버튼

빈 목록일 때 "등록된 상품이 없습니다" 표시.
```

### 댓글 컴포넌트

```
게시글 상세보기에 댓글 기능을 추가해줘.

테이블: tb_comment
필드: board_id, user_id, content, reg_date

표시 내용:
- 작성자 이름
- 댓글 내용
- 작성 시간 (yyyy-MM-dd HH:mm 포맷)
- 본인 댓글만 삭제 버튼

로그인한 사용자만 댓글 작성 가능.
```

### 프로필 카드

```
사용자 프로필 카드 컴포넌트를 만들어줘.

위치: html/member/profile_card.html

표시 내용:
- 프로필 이미지 (없으면 기본 이미지)
- 이름
- 이메일
- 가입일 (yyyy-MM-dd)
- 레벨 뱃지 (1-8: 일반, 9: 관리자)

Bootstrap Card 사용.
```

## 체크리스트

컴포넌트 개발 시 확인사항:

- [ ] setVar와 HTML의 변수명이 일치하는가?
- [ ] setLoop 후 HTML에서 올바른 이름으로 참조하는가?
- [ ] 날짜 필드를 포맷팅했는가?
- [ ] 빈 목록에 대한 처리를 했는가? ({#empty})
- [ ] 조건문 닫기 태그를 누락하지 않았는가?
- [ ] 폼 스크립트를 출력했는가? ({form_script})
- [ ] HTML 특수문자가 자동으로 escape되는가? (XSS 방지)

## 자주 하는 실수

### 1. next() 호출 누락

```jsp
// ❌ 잘못된 코드
DataSet info = user.findById(userId);
p.setVar("info", info);  // 데이터 없음!

// ✅ 올바른 코드
DataSet info = user.findById(userId);
info.next();  // 첫 행으로 이동
p.setVar("info", info);
```

### 2. 변수명 불일치

```jsp
// JSP
p.setLoop("boardList", list);

// HTML - ❌ 틀림
{#loop list}
    {list.title}
{#/loop}

// HTML - ✅ 올바름
{#loop boardList}
    {boardList.title}
{#/loop}
```

### 3. 조건문 닫기 누락

```html
<!-- ❌ 잘못된 코드 -->
{#if isLogin}
    <p>로그인됨</p>
<!-- {#/if} 누락! -->

<!-- ✅ 올바른 코드 -->
{#if isLogin}
    <p>로그인됨</p>
{#/if}
```

### 4. 반복 내 변수 오류

```html
<!-- ❌ 잘못된 코드 -->
{#loop list}
    <li>{title}</li>  <!-- list. 누락 -->
{#/loop}

<!-- ✅ 올바른 코드 -->
{#loop list}
    <li>{list.title}</li>
{#/loop}
```

## 관련 문서

- [페이지 및 라우팅 개발](malgn-pages-routing.md)
- [데이터베이스 작업](malgn-database.md)
- [상태 관리](malgn-state.md)
- [코딩 규칙](coding-rules.md)
