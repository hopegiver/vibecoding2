# 맑은프레임워크 - 배포 및 운영

## 1. 빌드

맑은프레임워크는 Ant로 DAO 클래스만 컴파일합니다. WAR 파일을 생성하지 않습니다.

**build.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="malgn-template" default="compile" basedir=".">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="public_html/WEB-INF/classes"/>
    <property name="lib.dir" value="public_html/WEB-INF/lib"/>

    <path id="classpath">
        <fileset dir="${lib.dir}">
            <include name="**/*.jar"/>
        </fileset>
    </path>

    <target name="init">
        <mkdir dir="${build.dir}"/>
    </target>

    <target name="compile" depends="init">
        <javac srcdir="${src.dir}" destdir="${build.dir}" includeantruntime="false" encoding="UTF-8" debug="true">
            <classpath refid="classpath"/>
        </javac>
    </target>

    <target name="clean">
        <delete dir="${build.dir}"/>
    </target>

    <target name="rebuild" depends="clean,compile"/>
</project>
```

**주요 명령어:**

| 명령어 | 설명 |
|--------|------|
| `ant compile` | DAO 클래스 컴파일 (`src/` → `WEB-INF/classes/`) |
| `ant clean` | 컴파일된 클래스 삭제 |
| `ant rebuild` | clean 후 다시 컴파일 |

## 2. web.xml

API 라우터만 매핑하는 단순한 구조입니다.

**public_html/WEB-INF/web.xml:**

```xml
<web-app>
  <display-name>malgn-template</display-name>
  <servlet>
    <servlet-name>APIRouter</servlet-name>
    <jsp-file>/api/index.jsp</jsp-file>
  </servlet>
  <servlet-mapping>
    <servlet-name>APIRouter</servlet-name>
    <url-pattern>/api/*</url-pattern>
  </servlet-mapping>
</web-app>
```

`/api/*` 경로의 모든 요청이 `/api/index.jsp` 라우터로 전달됩니다.

## 3. 데이터베이스 설정

### 스키마 실행

프로젝트 루트의 `schema.sql`을 MySQL에서 실행합니다.

```bash
mysql -u root -p mydb < schema.sql
```

### JNDI 설정

**public_html/WEB-INF/config.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <env>
        <jndi>jdbc/malgn</jndi>
    </env>
</config>
```

Tomcat의 `context.xml`에 대응하는 JNDI DataSource를 설정합니다.

```xml
<Context>
    <Resource name="jdbc/malgn"
              auth="Container"
              type="javax.sql.DataSource"
              driverClassName="com.mysql.cj.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/mydb"
              username="root"
              password="password"
              maxTotal="50"
              maxIdle="10"
              maxWaitMillis="10000"/>
</Context>
```

## 4. Tomcat 배포

### 방법 A: webapps 디렉토리 복사

프로젝트 폴더 자체(또는 `public_html`)를 Tomcat의 `webapps/` 아래에 복사하거나 심볼릭 링크합니다.

```
webapps/
  my-project/       ← public_html 내용
    WEB-INF/
      config.xml
      web.xml
      lib/malgn.jar
      classes/       ← ant compile 결과
    init.jsp
    index.jsp
    ...
```

### 방법 B: Context 설정 (권장)

Tomcat의 `server.xml` 또는 `conf/Catalina/localhost/`에 Context를 설정하면 프로젝트를 이동하지 않아도 됩니다.

```xml
<Context path="/my-project" docBase="/path/to/my-project/public_html" reloadable="true"/>
```

### 배포 후 확인

```
http://localhost:8080/my-project/
```

## 5. 배포 흐름 요약

```
1. schema.sql 실행          → DB 테이블 생성
2. config.xml JNDI 설정      → DB 연결 정보
3. ant compile               → DAO 클래스 컴파일
4. Tomcat Context 설정       → 프로젝트 경로 매핑
5. Tomcat 시작/재시작         → 서비스 확인
```

## 6. 운영 참고

### 스키마 변경

테이블 추가/변경 시 `schema.sql`을 수정하고 ALTER 문을 직접 실행합니다.

```sql
-- 예: 컬럼 추가
ALTER TABLE tb_board ADD COLUMN comment_count INT DEFAULT 0;
```

### 쿼리 최적화

N+1 문제가 발생하면 JOIN으로 전환합니다.

```java
// AS-IS: N+1
DataSet boards = dao.find("tb_board", "ORDER BY id DESC LIMIT 10");
while(boards.next()) {
    DataSet user = dao.findById("tb_user", boards.i("user_id"));
}

// TO-BE: JOIN
String sql = "SELECT b.*, u.name as user_name FROM tb_board b " +
             "LEFT JOIN tb_user u ON b.user_id = u.id ORDER BY b.id DESC LIMIT 10";
DataSet boards = dao.executeQuery(sql);
```

### 배포 체크리스트

- [ ] `schema.sql`이 최신 상태인가?
- [ ] `config.xml` JNDI 이름이 올바른가?
- [ ] `ant compile`이 오류 없이 완료되는가?
- [ ] Tomcat Context 설정이 올바른가?

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [페이지 및 라우팅](malgn-pages-routing.md)
- [API 개발 및 연동](malgn-api.md)
- [데이터베이스 작업](malgn-database.md)

---

[← 목차로 돌아가기](../_sidebar.md)
