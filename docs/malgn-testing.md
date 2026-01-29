# 맑은프레임워크 - 테스트 작성

맑은프레임워크 프로젝트의 테스트 작성 방법과 디버깅 전략을 학습합니다.

## 테스트 환경 설정

### JUnit 설정

**build.xml에 JUnit 추가:**

```xml
<project name="myproject" default="compile">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="build"/>
    <property name="test.dir" value="test"/>
    <property name="lib.dir" value="public_html/WEB-INF/lib"/>

    <path id="classpath">
        <fileset dir="${lib.dir}">
            <include name="**/*.jar"/>
        </fileset>
        <pathelement path="${build.dir}"/>
    </path>

    <target name="test" depends="compile">
        <junit printsummary="yes" haltonfailure="no">
            <classpath refid="classpath"/>
            <batchtest>
                <fileset dir="${test.dir}">
                    <include name="**/*Test.java"/>
                </fileset>
            </batchtest>
        </junit>
    </target>
</project>
```

### 프로젝트 구조

```
myproject/
├── src/
│   └── dao/
│       ├── UserDao.java
│       └── BoardDao.java
├── test/
│   └── dao/
│       ├── UserDaoTest.java
│       └── BoardDaoTest.java
└── public_html/
    └── WEB-INF/
        └── lib/
            ├── malgn.jar
            └── junit-4.13.jar
```

## DAO 단위 테스트

### 기본 테스트 구조

**test/dao/UserDaoTest.java:**

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
        // 테스트 데이터 초기화
    }

    @After
    public void tearDown() {
        // 테스트 데이터 정리
    }

    @Test
    public void testFindById() {
        DataSet user = userDao.findById(1);
        assertTrue(user.next());
        assertEquals("test@example.com", user.s("email"));
    }

    @Test
    public void testCreateUser() {
        DataSet newUser = userDao.create();
        newUser.put("email", "new@example.com");
        newUser.put("passwd", "hashed_password");
        newUser.put("name", "테스트");
        newUser.put("reg_date", "20260129120000");

        userDao.save(newUser);

        assertTrue(newUser.i("id") > 0);
    }

    @Test
    public void testIsDuplicateEmail() {
        boolean isDuplicate = userDao.isDuplicateEmail("test@example.com");
        assertTrue(isDuplicate);

        boolean notDuplicate = userDao.isDuplicateEmail("notexist@example.com");
        assertFalse(notDuplicate);
    }

    @Test
    public void testCheckLogin() {
        String email = "test@example.com";
        String passwd = "correct_hash";

        DataSet user = userDao.checkLogin(email, passwd);
        assertTrue(user.next());

        // 잘못된 비밀번호
        DataSet invalid = userDao.checkLogin(email, "wrong_hash");
        assertFalse(invalid.next());
    }
}
```

### CRUD 테스트

**test/dao/BoardDaoTest.java:**

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
        DataSet newBoard = boardDao.create();
        newBoard.put("user_id", 1);
        newBoard.put("title", "테스트 제목");
        newBoard.put("content", "테스트 내용");
        newBoard.put("reg_date", "20260129120000");
        boardDao.save(newBoard);

        testBoardId = newBoard.i("id");
    }

    @After
    public void tearDown() {
        // 테스트 게시글 삭제
        if(testBoardId > 0) {
            boardDao.delete(testBoardId);
        }
    }

    @Test
    public void testCreate() {
        assertTrue(testBoardId > 0);
    }

    @Test
    public void testFindById() {
        DataSet board = boardDao.findById(testBoardId);
        assertTrue(board.next());
        assertEquals("테스트 제목", board.s("title"));
        assertEquals("테스트 내용", board.s("content"));
    }

    @Test
    public void testUpdate() {
        DataSet board = boardDao.findById(testBoardId);
        assertTrue(board.next());

        board.put("title", "수정된 제목");
        board.put("content", "수정된 내용");
        boardDao.save(board);

        DataSet updated = boardDao.findById(testBoardId);
        assertTrue(updated.next());
        assertEquals("수정된 제목", updated.s("title"));
    }

    @Test
    public void testDelete() {
        boolean result = boardDao.delete(testBoardId);
        assertTrue(result);

        DataSet deleted = boardDao.findById(testBoardId);
        assertFalse(deleted.next());

        testBoardId = 0;  // tearDown에서 다시 삭제 안 하도록
    }

    @Test
    public void testFindByUserId() {
        DataSet list = boardDao.findByUserId(1, "ORDER BY id DESC");
        int count = 0;
        while(list.next()) {
            assertEquals(1, list.i("user_id"));
            count++;
        }
        assertTrue(count > 0);
    }

    @Test
    public void testIncreaseViewCount() {
        DataSet before = boardDao.findById(testBoardId);
        before.next();
        int beforeCount = before.i("view_count");

        boardDao.increaseViewCount(testBoardId);

        DataSet after = boardDao.findById(testBoardId);
        after.next();
        int afterCount = after.i("view_count");

        assertEquals(beforeCount + 1, afterCount);
    }
}
```

## 통합 테스트

### JSP 통합 테스트

**test/integration/BoardIntegrationTest.java:**

```java
package integration;

import org.junit.*;
import static org.junit.Assert.*;
import dao.*;
import malgnsoft.db.*;
import malgnsoft.util.*;

public class BoardIntegrationTest {
    private UserDao userDao;
    private BoardDao boardDao;
    private int testUserId;
    private int testBoardId;

    @Before
    public void setUp() {
        userDao = new UserDao();
        boardDao = new BoardDao();

        // 테스트 사용자 생성
        DataSet user = userDao.create();
        user.put("email", "test@example.com");
        user.put("passwd", Malgn.sha256("password123"));
        user.put("name", "테스터");
        user.put("reg_date", Malgn.time("yyyyMMddHHmmss"));
        userDao.save(user);
        testUserId = user.i("id");
    }

    @After
    public void tearDown() {
        // 테스트 데이터 정리
        if(testBoardId > 0) {
            boardDao.delete(testBoardId);
        }
        if(testUserId > 0) {
            userDao.delete(testUserId);
        }
    }

    @Test
    public void testBoardCreateFlow() {
        // 게시글 작성
        DataSet board = boardDao.create();
        board.put("user_id", testUserId);
        board.put("title", "통합 테스트");
        board.put("content", "통합 테스트 내용");
        board.put("view_count", 0);
        board.put("reg_date", Malgn.time("yyyyMMddHHmmss"));
        boardDao.save(board);
        testBoardId = board.i("id");

        // 조회
        DataSet info = boardDao.findById(testBoardId);
        assertTrue(info.next());
        assertEquals("통합 테스트", info.s("title"));

        // 조회수 증가
        boardDao.increaseViewCount(testBoardId);
        DataSet updated = boardDao.findById(testBoardId);
        updated.next();
        assertEquals(1, updated.i("view_count"));

        // 수정
        updated.put("title", "수정된 제목");
        boardDao.save(updated);
        DataSet modified = boardDao.findById(testBoardId);
        modified.next();
        assertEquals("수정된 제목", modified.s("title"));

        // 삭제
        boolean deleted = boardDao.delete(testBoardId);
        assertTrue(deleted);
        testBoardId = 0;
    }
}
```

## 테스트 데이터 준비

### 테스트 스키마

**schema_test.sql:**

```sql
-- 테스트 데이터베이스
CREATE DATABASE IF NOT EXISTS myproject_test;
USE myproject_test;

-- 사용자 테이블
CREATE TABLE tb_user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    passwd VARCHAR(64) NOT NULL,
    name VARCHAR(50) NOT NULL,
    reg_date VARCHAR(14) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 게시판 테이블
CREATE TABLE tb_board (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    view_count INT DEFAULT 0,
    reg_date VARCHAR(14) NOT NULL,
    mod_date VARCHAR(14),
    FOREIGN KEY (user_id) REFERENCES tb_user(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 테스트 사용자
INSERT INTO tb_user (email, passwd, name, reg_date) VALUES
('test@example.com', SHA2('password123', 256), '테스터', '20260129120000');

-- 테스트 게시글
INSERT INTO tb_board (user_id, title, content, view_count, reg_date) VALUES
(1, '테스트 게시글 1', '내용 1', 0, '20260129120000'),
(1, '테스트 게시글 2', '내용 2', 5, '20260129130000');
```

### 테스트 설정 분리

**public_html/WEB-INF/config_test.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <database>
        <driver>com.mysql.cj.jdbc.Driver</driver>
        <url>jdbc:mysql://localhost:3306/myproject_test</url>
        <username>root</username>
        <password>password</password>
    </database>
</config>
```

## Claude Code와 함께 테스트하기

### 테스트 작성 요청

```
UserDao의 단위 테스트를 작성해줘.

테스트 메서드:
- testFindById: ID로 조회
- testFindByEmail: 이메일로 조회
- testIsDuplicateEmail: 중복 체크
- testCheckLogin: 로그인 검증
- testCreate: 사용자 생성
- testUpdate: 사용자 수정
- testDelete: 사용자 삭제

JUnit 4 사용, @Before/@After로 초기화/정리.
```

### 테스트 실행 요청

```
BoardDaoTest를 실행해줘.

실행 방법:
1. 테스트 DB 초기화 (schema_test.sql)
2. ant test 실행
3. 결과 확인

실패한 테스트가 있으면 원인 분석 후 수정.
```

### 통합 테스트 요청

```
게시판 CRUD 통합 테스트를 작성해줘.

시나리오:
1. 테스트 사용자 생성
2. 게시글 작성
3. 게시글 조회 및 조회수 증가
4. 게시글 수정
5. 게시글 삭제
6. 테스트 데이터 정리

각 단계마다 assert로 검증.
```

## 디버깅 전략

### 로그 출력

**JSP에서 디버깅:**

```jsp
// 콘솔 출력 (Tomcat 로그)
System.out.println("boardId: " + boardId);
System.out.println("userId: " + userId);

// 파라미터 확인
System.out.println("title: " + f.get("title"));

// DataSet 내용 확인
while(list.next()) {
    System.out.println("id: " + list.i("id") + ", title: " + list.s("title"));
}
```

### 오류 추적

```jsp
try {
    BoardDao board = new BoardDao();
    DataSet info = board.findById(boardId);
    // ...
} catch(Exception e) {
    System.out.println("오류 발생: " + e.getMessage());
    e.printStackTrace();
}
```

### SQL 로그

**config.xml에 로그 설정:**

```xml
<config>
    <database>
        <driver>com.mysql.cj.jdbc.Driver</driver>
        <url>jdbc:mysql://localhost:3306/mydb?logger=Slf4JLogger&amp;profileSQL=true</url>
        <username>root</username>
        <password>password</password>
    </database>
    <debug>true</debug>
</config>
```

## 실전 프롬프트 예시

### DAO 테스트 생성

```
CommentDao의 단위 테스트를 만들어줘.

테스트 대상:
- findById
- findByBoardId
- create
- delete

@Before에서 테스트 댓글 생성,
@After에서 정리.

JUnit 4 사용.
```

### API 테스트 추가

```
/api/board_create.jsp API 테스트를 만들어줘.

테스트 케이스:
1. 정상 작성 (성공)
2. 제목 누락 (실패)
3. 내용 누락 (실패)
4. 로그인 안 됨 (실패)

HttpURLConnection 사용해서 실제 HTTP 요청.
```

### 성능 테스트

```
게시판 목록 조회 성능 테스트를 만들어줘.

시나리오:
- 1000개 게시글 생성
- 10페이지 조회 (페이지당 10개)
- 소요 시간 측정
- 1초 이내 완료 assert

테스트 후 데이터 정리.
```

## 체크리스트

테스트 작성 시 확인사항:

- [ ] @Before에서 테스트 데이터를 초기화했는가?
- [ ] @After에서 테스트 데이터를 정리했는가?
- [ ] 각 테스트 메서드는 독립적인가?
- [ ] 테스트 DB를 별도로 사용하는가?
- [ ] assert로 결과를 명확히 검증하는가?
- [ ] 예외 상황도 테스트했는가?
- [ ] 테스트 실행 순서에 의존하지 않는가?

## 자주 하는 실수

### 1. tearDown 누락

```java
// ❌ 잘못된 코드
@After
public void tearDown() {
    // 정리 코드 없음
}
// 테스트 데이터가 DB에 남음

// ✅ 올바른 코드
@After
public void tearDown() {
    if(testBoardId > 0) {
        boardDao.delete(testBoardId);
    }
}
```

### 2. 테스트 간 의존성

```java
// ❌ 잘못된 코드
@Test
public void test1_create() {
    // testBoardId 생성
}

@Test
public void test2_update() {
    // test1의 testBoardId 사용 (의존)
}

// ✅ 올바른 코드
@Before
public void setUp() {
    // 모든 테스트에서 사용할 데이터 생성
}
```

### 3. 프로덕션 DB 사용

```java
// ❌ 잘못된 코드 (위험!)
// config.xml 그대로 사용 (프로덕션 DB)

// ✅ 올바른 코드
// config_test.xml 사용 (테스트 DB)
```

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [데이터베이스 작업](malgn-database.md)
- [배포 및 운영](malgn-deployment.md)
- [코딩 규칙](coding-rules.md)
