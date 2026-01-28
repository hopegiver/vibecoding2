# 맑은프레임워크 .claude 설정 예제

## 개요

이 문서는 맑은프레임워크 프로젝트를 위한 `.claude` 폴더 설정 예제를 제공합니다. 실제 운영 중인 맑은프레임워크 프로젝트의 설정과 [코딩 원칙 문서](../malgn-template/docs/coding-principles.md)를 기반으로 작성되었습니다.

## 프로젝트 구조

```
your-malgn-project/
├── .claude/
│   └── rules/
│       ├── core-principles.md     # 핵심 원칙
│       └── coding-rules.md        # 코딩 규칙
├── CLAUDE.md                       # 프로젝트 컨텍스트 (선택)
├── public_html/
│   ├── WEB-INF/
│   │   └── config.xml              # 프레임워크 설정
│   ├── init.jsp                    # 전역 초기화
│   ├── main/                       # JSP 페이지
│   └── html/                       # HTML 템플릿
└── src/
    └── dao/                        # DAO 클래스
```

## 1. `.claude/rules/core-principles.md`

맑은프레임워크의 핵심 설계 철학을 설명합니다.

```markdown
# 맑은프레임워크 핵심 원칙

## ⚡ 절대 원칙 (이것만은 반드시 지켜라)

### 1. JSP와 HTML 완전 분리

**JSP는 로직만, HTML은 출력만**

❌ **절대 금지:**
\`\`\`jsp
<%
while(list.next()) {
%>
    <div class="user">
        <h3><%= list.s("name") %></h3>
    </div>
<%
}
%>
\`\`\`

✅ **반드시 이렇게:**

JSP 파일 (`/main/user_list.jsp`):
\`\`\`jsp
<%@ include file="/init.jsp" %><%
UserDao user = new UserDao();
DataSet list = user.find();

p.setLayout("default");
p.setBody("main.user_list");
p.setLoop("users", list);
p.display();
%>
\`\`\`

HTML 파일 (`/html/main/user_list.html`):
\`\`\`html
<!--@loop(users)-->
<div class="user">
    <h3>{{users.name}}</h3>
</div>
<!--/loop(users)-->
\`\`\`

**이유:**
- 디자이너는 HTML만, 개발자는 Java만 집중
- 로직 변경 시 HTML 수정 불필요
- HTML 변경 시 로직 수정 불필요

### 2. try-catch 사용 금지

**프레임워크가 모든 예외를 처리합니다**

❌ **절대 금지:**
\`\`\`jsp
<%
try {
    user.insert();
    m.p("성공");
} catch(Exception e) {
    m.p("오류: " + e.getMessage());
}
%>
\`\`\`

✅ **반드시 이렇게:**
\`\`\`jsp
<%
if(user.insert()) {
    m.p("성공");
} else {
    m.p("오류: " + user.getErrMsg());
}
%>
\`\`\`

**동작 방식:**
- boolean 리턴값으로 성공/실패 판단
- `getErrMsg()`로 에러 메시지 확인
- 모든 예외는 자동으로 로그 파일에 기록

### 3. 템플릿에 로직 넣지 말 것

**템플릿은 출력 전용**

❌ **절대 금지:**
\`\`\`html
<option {{status == 'active' ? 'selected' : ''}}>활성</option>
<div>{{ price * quantity }}</div>
<span>{{ user.getName().toUpperCase() }}</span>
\`\`\`

✅ **반드시 이렇게:**

JSP에서 로직 처리:
\`\`\`jsp
<%
p.setVar("selected", status.equals("active") ? "selected" : "");
p.setVar("total", price * quantity);
p.setVar("userName", user.getName().toUpperCase());
%>
\`\`\`

템플릿은 출력만:
\`\`\`html
<option {{selected}}>활성</option>
<div>{{total}}</div>
<span>{{userName}}</span>
\`\`\`

## 데이터 처리 원칙

### 1. GET/POST 파라미터 구분

**보안을 위해 반드시 다른 메소드 사용**

✅ **GET 파라미터: m.rs(), m.ri()** (XSS 필터 자동)
\`\`\`jsp
<%
String keyword = m.rs("keyword");  // 검색어
int page = m.ri("page");           // 페이지 번호
int id = m.ri("id");               // 조회 ID
%>
\`\`\`

✅ **POST 파라미터: f.get()** (원본 데이터)
\`\`\`jsp
<%
if(m.isPost()) {
    user.item("name", f.get("name"));
    user.item("content", f.get("content"));  // HTML 에디터 내용
}
%>
\`\`\`

**이유:**
- GET 파라미터는 URL에 노출되므로 XSS 필터 필요
- POST 데이터는 DB에 저장할 원본 (HTML 에디터 내용 등)

### 2. DataSet 사용 전 반드시 next() 호출

**커서를 첫 레코드로 이동**

❌ **금지:**
\`\`\`jsp
<%
UserDao user = new UserDao();
DataSet info = user.get(123);
String name = info.s("name");  // 에러 또는 빈 값!
%>
\`\`\`

✅ **올바름:**
\`\`\`jsp
<%
UserDao user = new UserDao();
DataSet info = user.get(123);

if(info.next()) {
    String name = info.s("name");
    m.p("이름: " + name);
} else {
    m.p("데이터를 찾을 수 없습니다.");
}
%>
\`\`\`

**이유:**
- DataSet의 커서는 초기에 -1 위치
- next()로 첫 번째 레코드로 이동 필요
- next()는 레코드 존재 여부도 확인 (boolean 리턴)

### 3. POST 처리 후 반드시 return

**이중 실행 방지**

❌ **금지:**
\`\`\`jsp
<%
if(m.isPost()) {
    user.insert();
    m.jsAlert("완료");
    // return 없음!
}
p.display();  // POST 후에도 실행됨
%>
\`\`\`

✅ **올바름:**
\`\`\`jsp
<%
if(m.isPost() && f.validate()) {
    user.insert();
    m.jsAlert("완료");
    m.jsReplace("list.jsp");
    return;  // 필수!
}
p.display();
%>
\`\`\`

## 페이지 개발 패턴

### Postback 패턴 (필수)

**등록/수정은 같은 JSP에서 처리**

\`\`\`jsp
<%@ include file="/init.jsp" %><%

// 유효성 검증 규칙 설정
f.addElement("name", null, "required:Y, minLen:2, maxLen:50");
f.addElement("email", null, "required:Y, email:Y");

// POST 처리 (등록)
if(m.isPost() && f.validate()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("reg_date", m.time());

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError("등록 실패: " + user.getErrMsg());
    }
    return;
}

// GET 처리 (폼 표시)
p.setLayout("default");
p.setBody("main.user_form");
p.setVar("is_insert", true);
p.setVar("form_script", f.getScript());  // 클라이언트 검증 스크립트
p.display();
%>
\`\`\`

**이유:**
- 한 파일에서 폼과 처리를 모두 담당
- 에러 발생 시 같은 페이지에서 재입력 가능
- 파일 수 감소로 프로젝트 구조 단순화

### Page 메소드 호출 순서

**반드시 이 순서로 호출**

\`\`\`jsp
<%
p.setLayout("default");         // 1. 레이아웃 설정 (선택)
p.setBody("main.content");      // 2. 본문 템플릿 (필수)
p.setVar("title", "제목");       // 3. 변수 설정
p.setLoop("list", dataSet);     // 4. 반복 데이터
p.display();                    // 5. 출력 (필수)
%>
\`\`\`

**이유:**
- 템플릿 파일을 먼저 지정해야 변수 바인딩 가능
- 순서를 지키지 않으면 변수가 치환되지 않음

## 데이터베이스 원칙

### 날짜는 VARCHAR + m.time()

**데이터베이스 벤더 종속성 제거**

❌ **금지: 데이터베이스별 함수**
\`\`\`jsp
<%
user.item("reg_date", "NOW()", "function");      // MySQL
user.item("reg_date", "SYSDATE", "function");    // Oracle
user.item("reg_date", "GETDATE()", "function");  // MSSQL
%>
\`\`\`

✅ **올바름: m.time() 사용**
\`\`\`jsp
<%
user.item("reg_date", m.time());                 // 20250124153045
user.item("reg_date", m.time("yyyyMMddHHmmss")); // 20250124153045
user.item("mod_date", m.time("yyyyMMdd"));       // 20250124
%>
\`\`\`

**테이블 정의:**
\`\`\`sql
CREATE TABLE tb_user (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    reg_date VARCHAR(14),   -- yyyyMMddHHmmss
    birth_date VARCHAR(8)   -- yyyyMMdd
);
\`\`\`

**이유:**
- 모든 데이터베이스에서 동일한 코드 사용
- 문자열 비교로 날짜 범위 검색 가능
- DB 마이그레이션 시 코드 수정 불필요

### 페이징이 필요 없으면 ListManager 사용 금지

**불필요한 COUNT 쿼리 실행 방지**

❌ **비효율:**
\`\`\`jsp
<%
// Excel 다운로드인데 ListManager 사용 (COUNT + SELECT 2번 실행)
ListManager lm = new ListManager();
lm.setTable("tb_user");
lm.setListNum(999999);
DataSet list = lm.getDataSet();
%>
\`\`\`

✅ **효율:**
\`\`\`jsp
<%
// 페이징 불필요 시 DataObject 사용 (SELECT 1번만 실행)
UserDao user = new UserDao();
DataSet list = user.find();
%>
\`\`\`
```

## 2. `.claude/rules/coding-rules.md`

구체적인 코딩 규칙과 패턴을 정의합니다.

```markdown
# 맑은프레임워크 코딩 규칙

## 필수 임포트

**모든 JSP 파일 상단에 include**

\`\`\`jsp
<%@ include file="/init.jsp" %><%
\`\`\`

**/public_html/init.jsp 내용:**
\`\`\`jsp
<%@ page import="java.util.*, java.io.*, malgnsoft.db.*, malgnsoft.util.*" %><%

Malgn m = new Malgn(request, response, out);
Form f = new Form(request);
Page p = new Page(request, response, out, pageContext);

// 인증 처리
Auth auth = new Auth(request, response);
int userId = 0;
String userName = "";

if(auth.isValid()) {
    userId = auth.getInt("user_id");
    userName = auth.getString("user_name");
}

p.setVar("userId", userId);
p.setVar("userName", userName);
%>
\`\`\`

## 폼 처리 패턴

### 1. 등록 페이지

\`\`\`jsp
<%@ include file="/init.jsp" %><%

// 유효성 검증 설정
f.addElement("name", null, "required:Y, minLen:2, maxLen:50");
f.addElement("email", null, "required:Y, email:Y");

// POST 처리
if(m.isPost() && f.validate()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("reg_date", m.time());

    if(user.insert()) {
        m.jsAlert("등록되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError(user.getErrMsg());
    }
    return;
}

// GET 처리
p.setLayout("default");
p.setBody("main.user_form");
p.setVar("is_insert", true);
p.setVar("form_script", f.getScript());
p.display();
%>
\`\`\`

### 2. 수정 페이지

\`\`\`jsp
<%@ include file="/init.jsp" %><%

int id = m.ri("id");

// 데이터 조회 (먼저!)
UserDao user = new UserDao();
DataSet info = user.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("데이터를 찾을 수 없습니다.");
    return;
}

// 유효성 검증 설정 (기존 값 설정)
f.addElement("name", info.s("name"), "required:Y");
f.addElement("email", info.s("email"), "required:Y, email:Y");

// POST 처리
if(m.isPost() && f.validate()) {
    user.item("name", f.get("name"));
    user.item("email", f.get("email"));
    user.item("mod_date", m.time());

    if(user.update("id = ?", new Object[]{id})) {
        m.jsAlert("수정되었습니다.");
        m.jsReplace("list.jsp");
    } else {
        m.jsError(user.getErrMsg());
    }
    return;
}

// GET 처리
p.setLayout("default");
p.setBody("main.user_form");
p.setVar("is_modify", true);
p.setVar("form_script", f.getScript());
p.display();
%>
\`\`\`

## 조회 및 목록 패턴

### 1. 목록 페이지 (페이징 O)

\`\`\`jsp
<%@ include file="/init.jsp" %><%

String keyword = m.rs("keyword");

ListManager lm = new ListManager();
lm.setRequest(request);
lm.setTable("tb_user");
lm.setListNum(20);

if(!"".equals(keyword)) {
    lm.addSearch("name,email", keyword, "LIKE");
}

lm.setOrderBy("id DESC");
DataSet list = lm.getDataSet();
String paging = lm.getPaging();

p.setLayout("default");
p.setBody("main.user_list");
p.setVar("keyword", keyword);
p.setVar("paging", paging);
p.setLoop("list", list);
p.display();
%>
\`\`\`

### 2. 목록 페이지 (페이징 X)

\`\`\`jsp
<%@ include file="/init.jsp" %><%

String keyword = m.rs("keyword");

UserDao user = new UserDao();

if(!"".equals(keyword)) {
    user.addSearch("name,email", keyword, "LIKE");
}

user.setOrderBy("id DESC");
DataSet list = user.find();

p.setLayout("default");
p.setBody("main.user_list");
p.setVar("keyword", keyword);
p.setLoop("list", list);
p.display();
%>
\`\`\`

### 3. 상세 페이지

\`\`\`jsp
<%@ include file="/init.jsp" %><%

int id = m.ri("id");

UserDao user = new UserDao();
DataSet info = user.find("id = ?", new Object[]{id});

if(!info.next()) {
    m.jsError("데이터를 찾을 수 없습니다.");
    return;
}

p.setLayout("default");
p.setBody("main.user_view");
p.setVar("name", info.s("name"));
p.setVar("email", info.s("email"));
p.setVar("reg_date", m.time("yyyy-MM-dd", info.s("reg_date")));
p.display();
%>
\`\`\`

## API 응답 패턴

### 1. JSON 응답 (AJAX)

\`\`\`jsp
<%@ include file="/init.jsp" %><%

Json j = new Json();

if(m.isPost()) {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
}
%>
\`\`\`

### 2. REST API

\`\`\`jsp
<%@ include file="/init.jsp" %><%

Json j = new Json();
RestAPI api = new RestAPI(request, response);

api.get(() -> {
    UserDao user = new UserDao();
    DataSet list = user.find();
    j.add("users", list);
    j.print();
});

api.post(() -> {
    UserDao user = new UserDao();
    user.item("name", f.get("name"));

    if(user.insert()) {
        j.success("등록되었습니다.", user.id);
    } else {
        j.error(user.getErrMsg());
    }
});
%>
\`\`\`

## 권한 체크 패턴

### 폴더별 init.jsp 활용

**/public_html/admin/init.jsp:**
\`\`\`jsp
<%@ include file="/init.jsp" %><%

// 관리자 권한 체크
if(userLevel < 9) {
    m.jsError("관리자 권한이 필요합니다.");
    m.jsReplace("/main/index.jsp");
    return;
}
%>
\`\`\`

**/public_html/admin/user_list.jsp:**
\`\`\`jsp
<%@ include file="init.jsp" %><%
// 이미 권한 체크됨

UserDao user = new UserDao();
DataSet list = user.find();

p.setLayout("admin");
p.setBody("admin.user_list");
p.setLoop("list", list);
p.display();
%>
\`\`\`

## 금지 사항

- ❌ JSP에 HTML 마크업 작성
- ❌ try-catch 사용
- ❌ 템플릿에 로직 넣기
- ❌ POST 후 return 누락
- ❌ DataSet에서 next() 호출 누락
- ❌ GET 파라미터를 f.get()으로 받기
- ❌ POST 데이터를 m.rs()로 받기
- ❌ AJAX 요청에서 jsReplace/redirect 사용
- ❌ 페이징 불필요한데 ListManager 사용
```

## 3. `CLAUDE.md` (선택)

간단한 프로젝트 개요만 제공합니다.

```markdown
# 맑은프레임워크 프로젝트

## 프로젝트 개요
맑은프레임워크 기반 웹 애플리케이션. JSP + MyBatis를 사용한 전통적인 Java 웹 애플리케이션으로, 템플릿 엔진을 통한 뷰 분리 패턴을 적용.

**기술 스택:** Java + JSP + MyBatis + 맑은프레임워크 1.14.0

## 핵심 원칙

1. ✅ JSP와 HTML 완전 분리
2. ✅ try-catch 사용 금지
3. ✅ POST 처리 후 반드시 return
4. ✅ GET은 m.rs()/m.ri(), POST는 f.get()
5. ✅ DataSet 사용 전 next() 호출
6. ✅ 날짜는 VARCHAR(14) + m.time()

## 파일 구조

\`\`\`
public_html/
├── init.jsp                # 전역 초기화
├── main/                   # 메인 JSP
│   ├── user_list.jsp
│   └── user_insert.jsp
└── html/                   # HTML 템플릿
    └── main/
        └── user_list.html

src/
└── dao/
    └── UserDao.java
\`\`\`

---
**개발 규칙:** `.claude/rules/` 폴더 참조
```

## 4. 실전 활용 예시

### 새로운 CRUD 페이지 추가 요청

```
프롬프트: "게시판 CRUD 페이지를 만들어줘. /board 경로에 list, insert, modify, delete 페이지를 추가해줘."
```

**Claude Code의 동작:**
1. `.claude/rules/core-principles.md` 읽고 JSP/HTML 분리 확인
2. `.claude/rules/coding-rules.md`에서 패턴 확인
3. 다음 파일들 생성:
   - `/public_html/board/board_list.jsp` (목록, ListManager 사용)
   - `/public_html/board/board_insert.jsp` (등록, Postback 패턴)
   - `/public_html/board/board_modify.jsp` (수정, Postback 패턴)
   - `/public_html/board/board_delete.jsp` (삭제 처리)
   - `/public_html/html/board/board_list.html` (목록 템플릿)
   - `/public_html/html/board/board_form.html` (등록/수정 공용 템플릿)
   - `/src/dao/BoardDao.java` (DAO 클래스)
4. 모든 파일이 규칙을 준수:
   - JSP에 HTML 없음
   - try-catch 없음
   - Postback 패턴 적용
   - 유효성 검증 포함

## 효과

이 `.claude` 설정으로:
- ✅ JSP와 HTML이 완전히 분리된 코드 생성
- ✅ try-catch 대신 boolean 체크로 예외 처리
- ✅ Postback 패턴을 자동으로 적용
- ✅ GET/POST 파라미터를 올바르게 구분
- ✅ DataSet 사용 시 항상 next() 호출
- ✅ 유효성 검증 자동 추가
- ✅ 날짜 처리를 VARCHAR + m.time()으로 통일

## 다음 단계

- [Pages .claude 설정 예제](pages-claude-setup.md)
- [Workers .claude 설정 예제](workers-claude-setup.md)
- [MCP 서버 설정 및 활용](mcp-setup.md)

---

[← 목차로 돌아가기](../_sidebar.md)
