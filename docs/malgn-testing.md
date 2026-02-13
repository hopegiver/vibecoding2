# 맑은프레임워크 - 테스트 및 디버깅

맑은프레임워크 프로젝트의 DAO 테스트 작성 방법과 디버깅 전략을 설명합니다.

## 테스트 환경 설정

### build.xml

```xml
<project name="malgn-template" default="compile" basedir=".">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="public_html/WEB-INF/classes"/>
    <property name="lib.dir" value="public_html/WEB-INF/lib"/>

    <path id="classpath">
        <fileset dir="${lib.dir}">
            <include name="**/*.jar"/>
        </fileset>
        <pathelement path="${build.dir}"/>
    </path>

    <target name="compile">
        <mkdir dir="${build.dir}"/>
        <javac srcdir="${src.dir}" destdir="${build.dir}" classpathref="classpath"
               encoding="UTF-8" includeantruntime="false"/>
    </target>

    <target name="clean">
        <delete dir="${build.dir}"/>
    </target>
</project>
```

### 설정 파일 (config.xml)

맑은프레임워크는 JNDI를 통해 데이터베이스에 연결합니다.

```xml
<config>
    <env>
        <jndi>jdbc/malgn</jndi>
    </env>
</config>
```

## DAO 단위 테스트

### 기본 CRUD 테스트

**BoardDaoTest.java:**

```java
package dao;

import org.junit.*;
import static org.junit.Assert.*;
import malgnsoft.db.*;

public class BoardDaoTest {
    private BoardDao boardDao;
    private int testBoardId;

    @Before
    public void setUp() {
        boardDao = new BoardDao();

        // 테스트 게시글 생성
        boardDao.item("user_id", 1);
        boardDao.item("title", "테스트 제목");
        boardDao.item("content", "테스트 내용");
        boardDao.item("reg_date", "20260129120000");
        boardDao.insert();

        // 방금 생성된 ID 조회
        boardDao.find("ORDER BY id DESC LIMIT 1");
        if (boardDao.next()) {
            testBoardId = boardDao.i("id");
        }
    }

    @After
    public void tearDown() {
        if (testBoardId > 0) {
            boardDao.execute("DELETE FROM tb_board WHERE id = " + testBoardId);
        }
    }

    @Test
    public void testCreate() {
        assertTrue(testBoardId > 0);
    }

    @Test
    public void testFindById() {
        boardDao.find("WHERE id = " + testBoardId);
        assertTrue(boardDao.next());
        assertEquals("테스트 제목", boardDao.s("title"));
        assertEquals("테스트 내용", boardDao.s("content"));
    }

    @Test
    public void testUpdate() {
        boardDao.item("title", "수정된 제목");
        boardDao.item("content", "수정된 내용");
        boardDao.update("id = " + testBoardId);

        boardDao.find("WHERE id = " + testBoardId);
        assertTrue(boardDao.next());
        assertEquals("수정된 제목", boardDao.s("title"));
    }

    @Test
    public void testDelete() {
        boardDao.execute("DELETE FROM tb_board WHERE id = " + testBoardId);

        boardDao.find("WHERE id = " + testBoardId);
        assertFalse(boardDao.next());

        testBoardId = 0; // tearDown에서 다시 삭제 안 하도록
    }

    @Test
    public void testFindList() {
        boardDao.find("WHERE user_id = 1 ORDER BY id DESC");
        int count = 0;
        while (boardDao.next()) {
            assertEquals(1, boardDao.i("user_id"));
            count++;
        }
        assertTrue(count > 0);
    }
}
```

### 사용자 DAO 테스트

**UserDaoTest.java:**

```java
package dao;

import org.junit.*;
import static org.junit.Assert.*;
import malgnsoft.db.*;

public class UserDaoTest {
    private UserDao userDao;

    @Before
    public void setUp() {
        userDao = new UserDao();
    }

    @Test
    public void testFindById() {
        userDao.find("WHERE id = 1");
        assertTrue(userDao.next());
        assertEquals("test@example.com", userDao.s("email"));
    }

    @Test
    public void testCreateUser() {
        userDao.item("email", "new@example.com");
        userDao.item("passwd", "hashed_password");
        userDao.item("name", "테스트");
        userDao.item("reg_date", "20260129120000");
        userDao.insert();

        userDao.find("WHERE email = 'new@example.com'");
        assertTrue(userDao.next());
        assertEquals("테스트", userDao.s("name"));

        // 정리
        userDao.execute("DELETE FROM tb_user WHERE email = 'new@example.com'");
    }
}
```

## 디버깅 전략

### 콘솔 로그 출력

**JSP에서 디버깅:**

```jsp
// 콘솔 출력 (Tomcat 로그)
System.out.println("boardId: " + boardId);
System.out.println("userId: " + userId);

// 파라미터 확인
System.out.println("title: " + f.get("title"));

// DataSet 조회 결과 확인
while (list.next()) {
    System.out.println("id: " + list.i("id") + ", title: " + list.s("title"));
}
```

### 오류 추적

```jsp
try {
    BoardDao board = new BoardDao();
    board.find("WHERE id = " + boardId);
    if (board.next()) {
        // 처리
    }
} catch (Exception e) {
    System.out.println("오류 발생: " + e.getMessage());
    e.printStackTrace();
}
```

## 자주 하는 실수

### 1. tearDown 누락

```java
// 잘못된 코드 - 테스트 데이터가 DB에 남음
@After
public void tearDown() {
    // 정리 코드 없음
}

// 올바른 코드
@After
public void tearDown() {
    if (testBoardId > 0) {
        boardDao.execute("DELETE FROM tb_board WHERE id = " + testBoardId);
    }
}
```

### 2. 잘못된 DAO 패턴

```java
// 잘못된 코드 - create()/save() 패턴은 없음
DataSet newBoard = boardDao.create();
newBoard.put("title", "테스트");
boardDao.save(newBoard);

// 올바른 코드 - item()/insert() 패턴 사용
boardDao.item("title", "테스트");
boardDao.item("user_id", 1);
boardDao.insert();
```

### 3. 테스트 간 의존성

```java
// 잘못된 코드 - 테스트 순서에 의존
@Test
public void test1_create() { /* testBoardId 생성 */ }
@Test
public void test2_update() { /* test1의 testBoardId 사용 */ }

// 올바른 코드 - @Before에서 각 테스트마다 데이터 생성
@Before
public void setUp() {
    // 모든 테스트에서 사용할 데이터 생성
}
```

## 체크리스트

- @Before에서 테스트 데이터를 초기화했는가?
- @After에서 테스트 데이터를 정리했는가?
- 각 테스트 메서드는 독립적인가?
- item()/insert() 패턴을 올바르게 사용했는가?
- assert로 결과를 명확히 검증하는가?

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [데이터베이스 작업](malgn-database.md)
- [배포 및 운영](malgn-deployment.md)
- [코딩 규칙](coding-rules.md)
