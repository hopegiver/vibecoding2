# 맑은프레임워크 - 페이지 및 라우팅 개발

맑은프레임워크의 페이지 구조와 라우팅 방식을 익히고, 실전에서 바로 사용할 수 있는 패턴을 학습합니다.

## 페이지 구조 기본

### JSP/HTML 분리 원칙

맑은프레임워크는 **JSP(로직)**와 **HTML(뷰)**을 완전히 분리합니다.

```
public_html/
├── member/
│   └── login.jsp          # 로직 (데이터 처리, 인증)
└── html/
    └── member/
        └── login.html     # 뷰 (HTML 템플릿)
```

**장점:**
- 디자이너와 개발자의 협업 용이
- 재사용 가능한 템플릿
- 유지보수 편리

## 기본 페이지 패턴

### 1. 단순 페이지

**JSP:** `public_html/main/about.jsp`

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

p.setLayout("main");
p.setBody("main.about");
p.setVar("title", "회사 소개");
p.display();

%>
```

**HTML:** `public_html/html/main/about.html`

```html
<div class="container mt-5">
    <h1>회사 소개</h1>
    <p>우리 회사는...</p>
</div>
```

### 2. 데이터 전달 페이지

**JSP:** `public_html/main/contact.jsp`

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

String companyName = "맑은소프트";
String phone = "010-1234-5678";
String email = "info@malgnsoft.com";

p.setLayout("main");
p.setBody("main.contact");
p.setVar("title", "연락처");
p.setVar("companyName", companyName);
p.setVar("phone", phone);
p.setVar("email", email);
p.display();

%>
```

**HTML:** `public_html/html/main/contact.html`

```html
<div class="container mt-5">
    <h1>연락처</h1>
    <dl class="row">
        <dt class="col-sm-3">회사명</dt>
        <dd class="col-sm-9">{companyName}</dd>

        <dt class="col-sm-3">전화</dt>
        <dd class="col-sm-9">{phone}</dd>

        <dt class="col-sm-3">이메일</dt>
        <dd class="col-sm-9">{email}</dd>
    </dl>
</div>
```

### 3. 파라미터 처리 페이지

**JSP:** `public_html/board/board_view.jsp`

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int boardId = m.ri("id");
if(boardId == 0) {
    m.jsAlert("잘못된 접근입니다.");
    m.jsReplace("/board/board_list.jsp");
    return;
}

BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    m.jsAlert("게시글을 찾을 수 없습니다.");
    m.jsReplace("/board/board_list.jsp");
    return;
}

// 조회수 증가
board.increaseViewCount(boardId);

// 날짜 포맷
info.put("reg_date_format", m.time("yyyy-MM-dd HH:mm", info.s("reg_date")));

p.setLayout("main");
p.setBody("board.board_view");
p.setVar("title", info.s("title"));
p.setVar("info", info);
p.display();

%>
```

**핵심:**
- `m.ri("id")`: GET 파라미터를 int로 받기 (XSS 자동 필터)
- `m.jsAlert()`: 자바스크립트 alert 출력
- `m.jsReplace()`: 페이지 이동
- `return` 필수: 이후 코드 실행 방지

**HTML:** `public_html/html/board/board_view.html`

```html
<div class="container mt-5">
    <h1>{info.title}</h1>
    <p class="text-muted">
        작성자: {info.user_name} | 작성일: {info.reg_date_format} | 조회수: {info.view_count}
    </p>
    <hr>
    <div class="content">
        {info.content}
    </div>
    <div class="mt-3">
        <a href="board_list.jsp" class="btn btn-secondary">목록</a>
        {#if info.user_id == userId}
        <a href="board_modify.jsp?id={info.id}" class="btn btn-primary">수정</a>
        <a href="board_delete.jsp?id={info.id}" class="btn btn-danger">삭제</a>
        {#/if}
    </div>
</div>
```

## 레이아웃 설정

### 레이아웃 종류

프로젝트에서 여러 레이아웃을 사용할 수 있습니다.

```
html/layout/
├── layout_main.html       # 기본 레이아웃
├── layout_admin.html      # 관리자 레이아웃
└── layout_simple.html     # 단순 레이아웃 (로그인 등)
```

### 레이아웃 전환

```jsp
// 기본 레이아웃
p.setLayout("main");

// 관리자 레이아웃
p.setLayout("admin");

// 단순 레이아웃 (로그인 페이지)
p.setLayout("simple");

// 레이아웃 없음 (API 응답 등)
// p.setLayout() 호출 안 함
```

### 레이아웃 템플릿 예제

**layout_simple.html:**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{title}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container">
        <div class="row justify-content-center mt-5">
            <div class="col-md-6">
                {__CONTENT__}
            </div>
        </div>
    </div>
</body>
</html>
```

## Postback 패턴

폼 제출과 화면 표시를 한 파일에서 처리하는 패턴입니다.

### 기본 구조

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// POST 요청 처리
if(m.isPost()) {
    // 폼 데이터 처리
    String title = f.get("title");
    String content = f.get("content");

    // 유효성 검사
    if(title.isEmpty()) {
        j.put("success", false);
        j.put("message", "제목을 입력하세요.");
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
    j.put("redirect", "/board/board_list.jsp");
    j.print();
    return;  // ⭐ 필수!
}

// GET 요청 - 화면 표시
f.addElement("title", null, "required");
f.addElement("content", null, "required");

p.setLayout("main");
p.setBody("board.board_write");
p.setVar("title", "게시글 작성");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**핵심 규칙:**
1. `if(m.isPost())` 먼저 처리
2. POST 처리 후 **반드시 return**
3. return 없으면 화면도 함께 렌더링됨 (오류)

## 페이지 이동 방법

### 1. 서버 사이드 리다이렉트

```jsp
// 즉시 이동
response.sendRedirect("/main/index.jsp");
return;

// 메시지 후 이동 (자바스크립트)
m.jsAlert("저장되었습니다.");
m.jsReplace("/board/board_list.jsp");
return;
```

### 2. 자바스크립트 이동 (AJAX 응답)

```jsp
// POST 처리 후
j.put("success", true);
j.put("message", "저장되었습니다.");
j.put("redirect", "/board/board_list.jsp");
j.print();
return;
```

**HTML (AJAX):**

```html
<form id="boardForm">
    <input type="text" name="title" class="form-control">
    <textarea name="content" class="form-control"></textarea>
    <button type="submit" class="btn btn-primary">저장</button>
</form>

<script>
document.getElementById('boardForm').addEventListener('submit', function(e) {
    e.preventDefault();

    fetch('board_write.jsp', {
        method: 'POST',
        body: new FormData(this)
    })
    .then(res => res.json())
    .then(data => {
        alert(data.message);
        if(data.success && data.redirect) {
            location.href = data.redirect;
        }
    });
});
</script>
```

## 파라미터 처리

### GET 파라미터

```jsp
// 문자열 (XSS 자동 필터)
String keyword = m.rs("keyword");

// 숫자
int page = m.ri("page", 1);  // 기본값 1
int boardId = m.ri("id");

// 여러 값 받기
String[] tags = request.getParameterValues("tags");
```

### POST 데이터

```jsp
// Form 객체 사용 (원본 보존)
String title = f.get("title");
String content = f.get("content");

// 배열
String[] categories = f.getArray("categories");
```

## 실전 프롬프트 예시

### 마이페이지 추가

```
마이페이지를 만들어줘.

경로: /member/mypage.jsp
레이아웃: main
표시 정보: 이름, 이메일, 가입일
수정 버튼 추가

로그인 안 되어 있으면 로그인 페이지로 이동.
```

### 검색 페이지 추가

```
상품 검색 페이지를 만들어줘.

경로: /product/search.jsp
파라미터: keyword (GET)
검색 대상: 상품명, 설명
페이징: 20개씩
정렬: 최신순

검색어가 없으면 전체 목록 표시.
```

### 폼 제출 페이지 추가

```
문의하기 페이지를 만들어줘.

경로: /main/inquiry.jsp
필드: name, email, subject, message
Postback 패턴 사용
AJAX 제출 (JSON 응답)

유효성 검사:
- 모든 필드 필수
- 이메일 형식 체크
```

## 체크리스트

페이지 개발 시 확인사항:

- [ ] JSP와 HTML이 분리되어 있는가?
- [ ] setLayout, setBody 호출했는가?
- [ ] setVar로 필요한 변수를 모두 전달했는가?
- [ ] Postback 패턴에서 return을 빠뜨리지 않았는가?
- [ ] 파라미터 검증을 했는가?
- [ ] 인증이 필요한 페이지에서 isLogin 체크했는가?
- [ ] 오류 시 적절한 메시지와 리다이렉트를 제공하는가?

## 자주 하는 실수

### 1. Postback 후 return 누락

```jsp
// ❌ 잘못된 코드
if(m.isPost()) {
    // 처리...
    j.print();
    // return 없음!
}
p.display();  // 화면도 렌더링되어 오류 발생
```

```jsp
// ✅ 올바른 코드
if(m.isPost()) {
    // 처리...
    j.print();
    return;  // 필수!
}
p.display();
```

### 2. 레이아웃 경로 오류

```jsp
// ❌ 잘못된 경로
p.setLayout("layout/layout_main");  // 틀림

// ✅ 올바른 경로
p.setLayout("main");  // html/layout/layout_main.html
```

### 3. 변수명 불일치

```jsp
// JSP
p.setVar("userName", "홍길동");

// HTML - ❌ 틀림
<p>{username}</p>

// HTML - ✅ 올바름
<p>{userName}</p>
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [컴포넌트 개발](malgn-components.md)
- [상태 관리](malgn-state.md)
- [코딩 규칙](coding-rules.md)
