# 맑은프레임워크 - 프로젝트 시작하기

맑은프레임워크로 새 프로젝트를 시작하는 방법을 단계별로 안내합니다.

## 전제조건

- Java JDK 8 이상
- Apache Tomcat 8 이상
- MySQL (또는 호환 DB)
- VSCode + Claude Code
- 맑은프레임워크 라이브러리 (malgn.jar)

## 1. 프로젝트 초기 설정

### Claude에게 요청하기

```
맑은프레임워크 프로젝트를 새로 만들어줘.
프로젝트 이름: myproject
주요 기능: 회원 시스템, 게시판

다음 구조로 생성:
- public_html/ (JSP 파일)
- src/dao/ (Java DAO)
- schema.sql (DB 스키마)
```

### 생성되는 기본 구조

```
myproject/
├── public_html/
│   ├── init.jsp                 # 공통 초기화
│   ├── index.jsp                # 루트 리다이렉트
│   ├── member/                  # 회원 기능
│   ├── main/                    # 메인 페이지
│   ├── html/                    # HTML 템플릿
│   ├── css/
│   ├── js/
│   └── WEB-INF/
│       ├── config.xml           # DB 설정
│       └── lib/
│           └── malgn.jar
├── src/
│   └── dao/                     # DAO 클래스
└── schema.sql                   # DB 스키마
```

## 2. init.jsp 설정

모든 JSP 파일에서 공통으로 사용하는 초기화 파일입니다.

### init.jsp 예제

```jsp
<%@ page import="java.util.*,java.io.*,malgnsoft.db.*,malgnsoft.util.*,malgnsoft.json.*,dao.*" %><%

Malgn m = new Malgn(request, response, out);

Form f = new Form();
f.setRequest(request);

Page p = new Page();
p.setRequest(request);
p.setPageContext(pageContext);
p.setWriter(out);

Json j = new Json(out);

// 인증 정보
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

%>
```

**주요 객체:**
- `m`: Malgn - 유틸리티 함수 (시간, 해시, 파라미터)
- `f`: Form - 폼 데이터 처리
- `p`: Page - 템플릿 엔진
- `j`: Json - JSON 응답
- `auth`: Auth - 인증/세션 관리

## 3. 데이터베이스 설정

### config.xml 설정

`public_html/WEB-INF/config.xml`에 DB 연결 정보를 설정합니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <database>
        <driver>com.mysql.cj.jdbc.Driver</driver>
        <url>jdbc:mysql://localhost:3306/mydb?useSSL=false&amp;serverTimezone=UTC</url>
        <username>root</username>
        <password>password</password>
    </database>
</config>
```

### 스키마 생성

```sql
-- schema.sql
CREATE TABLE tb_user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    passwd VARCHAR(64) NOT NULL,
    name VARCHAR(50) NOT NULL,
    reg_date VARCHAR(14) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**날짜 필드**: VARCHAR(14) 형식 사용 (YYYYMMDDHHmmss)

### Claude에게 스키마 생성 요청

```
schema.sql을 읽고 MySQL에 테이블을 생성해줘.
DB 정보는 config.xml 참고.
```

## 4. 첫 페이지 만들기

### 메인 페이지 JSP

`public_html/main/index.jsp`:

```jsp
<%@ page contentType="text/html; charset=utf-8" %><%@ include file="/init.jsp" %><%

p.setLayout("main");
p.setBody("main.index");
p.setVar("title", "홈페이지");
p.display();

%>
```

**패턴 설명:**
1. `contentType` 설정
2. `init.jsp` include
3. `p.setLayout()` - 레이아웃 설정
4. `p.setBody()` - 본문 템플릿 설정
5. `p.setVar()` - 변수 전달
6. `p.display()` - 렌더링

### HTML 템플릿

`public_html/html/main/index.html`:

```html
<div class="container mt-5">
    <h1>환영합니다!</h1>
    <p>맑은프레임워크로 만든 웹사이트입니다.</p>

    {#if isLogin}
    <p>안녕하세요, {userName}님!</p>
    {#else}
    <p><a href="/member/login.jsp">로그인</a>하세요.</p>
    {#/if}
</div>
```

### 레이아웃 템플릿

`public_html/html/layout/layout_main.html`:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{title}</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="/css/style.css" rel="stylesheet">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
        <div class="container">
            <a class="navbar-brand" href="/">My Site</a>
            <ul class="navbar-nav ms-auto">
                {#if isLogin}
                <li class="nav-item"><span class="nav-link">{userName}</span></li>
                <li class="nav-item"><a class="nav-link" href="/member/logout.jsp">로그아웃</a></li>
                {#else}
                <li class="nav-item"><a class="nav-link" href="/member/login.jsp">로그인</a></li>
                {#/if}
            </ul>
        </div>
    </nav>

    {__CONTENT__}

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

**핵심:**
- `{__CONTENT__}`: 본문이 삽입되는 위치
- `{변수명}`: JSP에서 setVar로 전달한 값
- `{#if}...{#/if}`: 조건문

## 5. 서버 실행 및 테스트

### Claude에게 요청

```
Tomcat 서버를 시작하고 http://localhost:8080 으로 접속해서 테스트해줘.
```

### 수동 실행 (참고)

```bash
# Tomcat 시작
catalina.bat start   # Windows
catalina.sh start    # Linux/Mac

# 브라우저에서 확인
# http://localhost:8080/myproject/
```

## 6. 실전 프롬프트 예시

### 회원가입 기능 추가

```
회원가입 기능을 추가해줘.

경로: /member/register.jsp
필드: email, passwd, name
DAO: UserDao (tb_user 테이블)

체크사항:
- 이메일 중복 확인
- SHA-256 암호화
- Postback 패턴 사용
- 성공 시 로그인 페이지로 리다이렉트
```

### 게시판 목록 추가

```
게시판 목록 기능을 추가해줘.

경로: /board/board_list.jsp
테이블: tb_board
페이징: 10개씩
검색: 제목+내용
정렬: 최신순

ListManager 사용해서 구현.
```

### API 엔드포인트 추가

```
게시글 좋아요 API를 추가해줘.

경로: /api/board_like.jsp
메서드: POST
파라미터: board_id
응답: JSON (success, message)

로그인 체크 필수.
중복 좋아요 방지 (tb_board_like 테이블).
```

## 7. 다음 단계

프로젝트 기본 구조가 완성되었습니다. 이제 다음 가이드를 참고하여 기능을 확장하세요:

- [페이지 및 라우팅 개발](malgn-pages-routing.md)
- [컴포넌트 개발](malgn-components.md)
- [API 개발 및 연동](malgn-api.md)
- [데이터베이스 작업](malgn-database.md)

## 체크리스트

프로젝트 시작 시 확인사항:

- [ ] init.jsp 설정 완료
- [ ] config.xml DB 설정 완료
- [ ] 스키마 생성 완료
- [ ] 메인 페이지 작동 확인
- [ ] 레이아웃 템플릿 정상 렌더링
- [ ] CSS/JS 파일 로드 확인
- [ ] 인증 객체 (auth) 동작 확인

## 문제 해결

### init.jsp import 오류

**증상:** `package malgnsoft does not exist`

**해결:**
```
WEB-INF/lib/malgn.jar 파일이 있는지 확인해줘.
없으면 맑은프레임워크 라이브러리를 다운로드해야 해.
```

### 템플릿 렌더링 오류

**증상:** 빈 화면 또는 `{변수명}` 그대로 표시

**해결:**
```
p.setLayout()과 p.setBody()의 경로를 확인해줘.
html/ 폴더에 해당 템플릿 파일이 있는지 체크.
```

### DB 연결 실패

**증상:** `Cannot load JDBC driver class`

**해결:**
```
WEB-INF/lib/mysql-connector-java.jar 파일을 추가해줘.
config.xml의 DB 연결 정보도 확인.
```

## 관련 문서

- [.claude 설정 예제](malgn-claude-setup.md)
- [프로젝트 구조 표준](project-structure.md)
- [코딩 규칙](coding-rules.md)
