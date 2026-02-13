# 맑은프레임워크 - 상태 관리

맑은프레임워크에서 세션, 쿠키, 파라미터를 활용한 상태 관리 방법을 학습합니다.

## 세션 관리 (Auth 클래스)

### 기본 인증 구조

**init.jsp에 기본 설정:**

```jsp
int userId = 0;
String userName = "";
boolean isLogin = false;

Auth auth = new Auth(request, response);
if(auth.isValid()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
    isLogin = true;
}

p.setVar("userId", userId);
p.setVar("userName", userName);
p.setVar("isLogin", isLogin);
```

### 로그인 처리

**member/login.jsp:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// POST 처리
if(m.isPost()) {
    String email = f.get("email");
    String passwd = f.get("passwd");

    // 유효성 검사
    if(email.isEmpty() || passwd.isEmpty()) {
        j.put("success", false);
        j.put("message", "이메일과 비밀번호를 입력하세요.");
        j.print();
        return;
    }

    // SHA-256 암호화
    String hashedPasswd = m.sha256(passwd);

    // DB 조회
    UserDao user = new UserDao();
    DataSet info = user.checkLogin(email, hashedPasswd);

    if(!info.next()) {
        j.put("success", false);
        j.put("message", "이메일 또는 비밀번호가 일치하지 않습니다.");
        j.print();
        return;
    }

    // 세션 저장
    Auth auth = new Auth(request, response);
    auth.set("user_id", info.i("id"));
    auth.set("user_name", info.s("name"));
    auth.set("user_email", info.s("email"));

    j.put("success", true);
    j.put("message", "로그인되었습니다.");
    j.put("redirect", "/main/index.jsp");
    j.print();
    return;
}

// GET - 화면 표시
f.addElement("email", null, "required|email");
f.addElement("passwd", null, "required");

p.setLayout("simple");
p.setBody("member.login");
p.setVar("title", "로그인");
p.setVar("form_script", f.getScript());
p.display();

%>
```

**UserDao.java:**

```java
public class UserDao extends DataObject {
    public UserDao() {
        this.table = "tb_user";
        this.PK = "id";
    }

    public DataSet checkLogin(String email, String passwd) {
        return this.find("email = ? AND passwd = ?", new Object[]{email, passwd});
    }
}
```

### 로그아웃 처리

**member/logout.jsp:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

Auth auth = new Auth(request, response);
auth.clear();

response.sendRedirect("/main/index.jsp");

%>
```

### 로그인 체크

**인증이 필요한 페이지:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// 로그인 체크
if(!isLogin) {
    m.jsError("로그인이 필요합니다.");
    m.jsReplace("/member/login.jsp");
    return;
}

// 이후 로직...
```

### 권한 체크

**게시글 수정 페이지:**

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

if(!isLogin) {
    m.jsError("로그인이 필요합니다.");
    m.jsReplace("/member/login.jsp");
    return;
}

int boardId = m.ri("id");
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    m.jsError("게시글을 찾을 수 없습니다.");
    m.jsReplace("/board/board_list.jsp");
    return;
}

// 권한 체크
if(info.i("user_id") != userId) {
    m.jsError("수정 권한이 없습니다.");
    m.jsReplace("/board/board_view.jsp?id=" + boardId);
    return;
}

// 이후 로직...
```

## 파라미터 처리

### GET 파라미터 (XSS 자동 필터)

```jsp
// 문자열 (XSS 필터링됨)
String keyword = m.rs("keyword");           // 빈 문자열 반환
String keyword = m.rs("keyword", "기본값"); // 기본값 지정

// 숫자
int page = m.ri("page");                    // 0 반환
int page = m.ri("page", 1);                 // 기본값 1

// 배열
String[] tags = request.getParameterValues("tags");
```

**XSS 필터링:**
- `<script>` → `&lt;script&gt;`
- `<img src=x onerror=alert(1)>` → 이스케이프 처리

### POST 데이터 (원본 보존)

```jsp
// Form 객체 사용 (원본 유지)
String title = f.get("title");
String content = f.get("content");

// 배열
String[] categories = f.getArray("categories");

// 파일
Upload upload = new Upload(request);
```

**GET vs POST:**
- **GET (m.rs):** XSS 자동 필터링 (검색어, 페이지 번호 등)
- **POST (f.get):** 원본 보존 (게시글 내용, 댓글 등)

### 파라미터 유효성 검사

```jsp
// 숫자 파라미터
int boardId = m.ri("id");
if(boardId == 0) {
    m.jsError("잘못된 접근입니다.");
    m.jsReplace("/board/board_list.jsp");
    return;
}

// 문자열 파라미터
String title = f.get("title");
if(title.isEmpty()) {
    j.put("success", false);
    j.put("message", "제목을 입력하세요.");
    j.print();
    return;
}

// 길이 체크
if(title.length() > 100) {
    j.put("success", false);
    j.put("message", "제목은 100자 이내로 입력하세요.");
    j.print();
    return;
}
```

## 쿠키 사용

### 쿠키 저장

```jsp
// 쿠키 생성 (7일)
Cookie cookie = new Cookie("remember_email", email);
cookie.setMaxAge(60 * 60 * 24 * 7);  // 초 단위
cookie.setPath("/");
response.addCookie(cookie);
```

### 쿠키 읽기

```jsp
String rememberEmail = "";

Cookie[] cookies = request.getCookies();
if(cookies != null) {
    for(Cookie cookie : cookies) {
        if(cookie.getName().equals("remember_email")) {
            rememberEmail = cookie.getValue();
            break;
        }
    }
}

p.setVar("rememberEmail", rememberEmail);
```

**HTML:**

```html
<form method="post">
    <input type="email" name="email" value="{{rememberEmail}}" class="form-control">
    <input type="password" name="passwd" class="form-control">
    <label>
        <input type="checkbox" name="remember" value="1"> 이메일 기억하기
    </label>
    <button type="submit">로그인</button>
</form>
```

### 쿠키 삭제

```jsp
Cookie cookie = new Cookie("remember_email", "");
cookie.setMaxAge(0);  // 즉시 삭제
cookie.setPath("/");
response.addCookie(cookie);
```

## Postback 패턴

### 기본 Postback

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

// POST 처리
if(m.isPost()) {
    String title = f.get("title");
    String content = f.get("content");

    // 유효성 검사
    if(title.isEmpty()) {
        j.put("success", false);
        j.put("message", "제목을 입력하세요.");
        j.print();
        return;
    }

    // 저장
    BoardDao board = new BoardDao();
    board.item("user_id", userId);
    board.item("title", title);
    board.item("content", content);
    board.item("reg_date", m.time("yyyyMMddHHmmss"));
    board.insert();

    j.put("success", true);
    j.put("message", "저장되었습니다.");
    j.put("redirect", "/board/board_list.jsp");
    j.print();
    return;  // 필수!
}

// GET - 화면 표시
f.addElement("title", null, "required");
f.addElement("content", null, "required");

p.setLayout("main");
p.setBody("board.board_write");
p.setVar("title", "게시글 작성");
p.setVar("form_script", f.getScript());
p.display();

%>
```

### 수정 페이지 Postback

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

int boardId = m.ri("id");
BoardDao board = new BoardDao();
DataSet info = board.findById(boardId);
if(!info.next()) {
    m.jsError("게시글을 찾을 수 없습니다.");
    m.jsReplace("/board/board_list.jsp");
    return;
}

// 권한 체크
if(info.i("user_id") != userId) {
    m.jsError("수정 권한이 없습니다.");
    m.jsReplace("/board/board_view.jsp?id=" + boardId);
    return;
}

// POST 처리
if(m.isPost()) {
    String title = f.get("title");
    String content = f.get("content");

    if(title.isEmpty()) {
        j.put("success", false);
        j.put("message", "제목을 입력하세요.");
        j.print();
        return;
    }

    // 수정
    board.item("title", f.get("title"));
    board.item("content", f.get("content"));
    board.item("mod_date", m.time("yyyyMMddHHmmss"));
    board.update("id = " + boardId);

    j.put("success", true);
    j.put("message", "수정되었습니다.");
    j.put("redirect", "/board/board_view.jsp?id=" + boardId);
    j.print();
    return;
}

// GET - 화면 표시
f.addElement("title", info.s("title"), "required");
f.addElement("content", info.s("content"), "required");

p.setLayout("main");
p.setBody("board.board_modify");
p.setVar("title", "게시글 수정");
p.setVar("info", info);
p.setVar("form_script", f.getScript());
p.display();

%>
```

## 자주 하는 실수

### 1. 로그인 체크 누락

```jsp
// 잘못된 코드
int boardId = m.ri("id");
BoardDao board = new BoardDao();
// 로그인 체크 없음!

// 올바른 코드
if(!isLogin) {
    m.jsError("로그인이 필요합니다.");
    m.jsReplace("/member/login.jsp");
    return;
}
```

### 2. 비밀번호 평문 저장

```jsp
// 잘못된 코드 (위험!)
String passwd = f.get("passwd");
user.put("passwd", passwd);  // 평문 저장

// 올바른 코드
String passwd = f.get("passwd");
String hashedPasswd = m.sha256(passwd);
user.put("passwd", hashedPasswd);
```

### 3. GET/POST 혼용

```jsp
// 잘못된 코드
String keyword = f.get("keyword");  // POST가 아님

// 올바른 코드
String keyword = m.rs("keyword");  // GET 파라미터
```

### 4. 쿠키 경로 미설정

```jsp
// 잘못된 코드
Cookie cookie = new Cookie("name", "value");
response.addCookie(cookie);  // 경로 없음

// 올바른 코드
Cookie cookie = new Cookie("name", "value");
cookie.setPath("/");  // 전체 경로에서 접근 가능
response.addCookie(cookie);
```

## 보안 고려사항

### 1. 세션 타임아웃

```jsp
// web.xml 설정
<session-config>
    <session-timeout>30</session-timeout>  <!-- 30분 -->
</session-config>
```

### 2. CSRF 방지

```jsp
// 폼에 토큰 추가
String csrfToken = m.sha256(session.getId() + m.time("yyyyMMddHHmmss"));
session.setAttribute("csrf_token", csrfToken);

p.setVar("csrfToken", csrfToken);
```

**HTML:**

```html
<form method="post">
    <input type="hidden" name="csrf_token" value="{{csrfToken}}">
    <!-- 폼 필드 -->
</form>
```

**POST 처리:**

```jsp
String submittedToken = f.get("csrf_token");
String sessionToken = (String)session.getAttribute("csrf_token");

if(!submittedToken.equals(sessionToken)) {
    j.put("success", false);
    j.put("message", "잘못된 요청입니다.");
    j.print();
    return;
}
```

### 3. SQL Injection 방지

```jsp
// 잘못된 코드
String email = f.get("email");
DataSet user = dao.find("email = '" + email + "'");

// 올바른 코드
String email = f.get("email");
DataSet user = dao.find("email = ?", new Object[]{email});
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [페이지 및 라우팅 개발](malgn-pages-routing.md)
- [보안 가이드라인](security.md)
- [코딩 규칙](coding-rules.md)
