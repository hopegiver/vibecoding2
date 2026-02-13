# 맑은프레임워크 - 프로젝트 시작하기

## 전제조건

- Java JDK 8 이상
- Apache Tomcat 8 이상 (또는 Resin)
- MySQL
- VSCode + Claude Code
- Ant (빌드 도구)

## 1. 프로젝트 클론

```bash
git clone https://github.com/hopegiver/malgn-template.git my-project
cd my-project
```

## 2. MCP 연결 확인

프로젝트에 `.mcp.json`이 포함되어 있어 MCP가 자동으로 설정됩니다. Claude Code에서 확인:

```
mcp 연결 테스트
```

7개 도구(get_context, validate_code, get_class, get_rules, get_pattern, get_doc, search_docs)가 모두 정상이면 준비 완료.

## 3. 데이터베이스 설정

1. MySQL에 데이터베이스 생성
2. `schema.sql` 실행하여 테이블 생성
3. `public_html/WEB-INF/config.xml`에 JNDI 설정

**config.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <jndi>jdbc/malgn</jndi>
    </env>
</config>
```

## 4. 프로젝트 구조 이해

```
my-project/
├── .claude/                # Claude Code 설정
│   ├── settings.json       # 권한 및 훅 설정
│   ├── rules/malgn.md      # 코딩 규칙 (MCP 참조)
│   └── hooks/post-write.sh # 자동 검증 훅
├── .mcp.json               # MCP 서버 설정
├── CLAUDE.md               # 프로젝트 컨텍스트
├── GUIDE.md                # 코딩 가이드 (상세)
├── build.xml               # Ant 빌드 스크립트
├── schema.sql              # DB 스키마
├── src/dao/                # DAO 클래스 (Java)
└── public_html/
    ├── init.jsp            # 공통 초기화
    ├── index.jsp           # 루트 리다이렉트
    ├── {기능}/              # JSP (로직)
    ├── html/               # HTML 템플릿 (출력)
    │   ├── layout/         # 레이아웃
    │   └── {기능}/
    ├── api/                # REST API
    │   ├── init.jsp        # API 초기화 (JWT, CORS)
    │   └── index.jsp       # API 라우터
    ├── assets/             # 정적 파일
    │   ├── css/
    │   └── js/common.js
    └── WEB-INF/
        ├── config.xml      # DB 설정
        ├── web.xml         # 서블릿 매핑
        └── lib/malgn.jar   # 프레임워크 라이브러리
```

## 5. 핵심 파일 이해

### init.jsp - 공통 초기화

모든 JSP에서 include하는 파일. 다음 객체가 자동 생성됩니다:

| 객체 | 클래스 | 용도 |
|------|--------|------|
| `m` | Malgn | 유틸리티 (시간, 해시, 파라미터) |
| `f` | Form | 폼 데이터 처리, 유효성 검증 |
| `p` | Page | 템플릿 엔진, 레이아웃 |
| `j` | Json | JSON 응답 |
| `auth` | Auth | 인증/세션 관리 |

자동 초기화되는 변수: `userId`, `userName`, `isLogin`

### 템플릿 문법

```html
{{변수명}}                    <!-- 변수 출력 -->
<!--@if(isLogin)-->           <!-- 조건문 -->
<!--/if(isLogin)-->
<!--@nif(isLogin)-->          <!-- 부정 조건문 -->
<!--/nif(isLogin)-->
<!--@loop(list)-->            <!-- 반복문 -->
  {{list.id}} {{list.name}}
<!--/loop(list)-->
<!--@include(BODY)-->         <!-- 레이아웃에서 본문 삽입 -->
```

## 6. 첫 CRUD 만들기

### 방법 A: 슬래시 커맨드 (권장)

```
/project:schema tb_notice 공지사항
```
→ 스키마 생성 후:
```
/project:crud tb_notice
```
→ DAO + JSP 5개 + HTML 4개 자동 생성

### 방법 B: 자연어

```
공지사항 게시판을 만들어줘. 테이블은 tb_notice이고 제목, 내용, 작성자, 조회수가 필요해.
```

## 7. 컴파일 & 확인

```bash
ant compile
```

WAS(Tomcat/Resin)를 재시작하고 브라우저에서 확인합니다.

## 자주 쓰는 커맨드

| 커맨드 | 설명 |
|--------|------|
| `/project:crud` | CRUD 전체 생성 |
| `/project:api` | REST API 생성 |
| `/project:new-page` | 단일 페이지 생성 |
| `/project:schema` | 테이블 스키마 생성 |
| `/project:validate` | 코드 규칙 검증 |
| `/project:review` | 코드 리뷰 |

## 자동 검증

JSP, HTML, Java(DAO) 파일을 작성하면 **Hook이 자동으로 규칙 위반을 체크**합니다. 위반이 발견되면 Claude가 자동으로 수정 제안을 합니다.

## 다음 단계

- [.claude 설정 예제](malgn-claude-setup.md)
- [페이지 및 라우팅 개발](malgn-pages-routing.md)
- [API 개발 및 연동](malgn-api.md)
- [데이터베이스 작업](malgn-database.md)

---

[← 목차로 돌아가기](../_sidebar.md)
