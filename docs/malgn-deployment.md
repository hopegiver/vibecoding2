# 맑은프레임워크 - 배포 및 운영

맑은프레임워크 프로젝트의 배포, 모니터링, 성능 최적화 방법을 학습합니다.

## WAR 파일 생성

### Ant 빌드 설정

**build.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="myproject" default="war" basedir=".">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="build"/>
    <property name="web.dir" value="public_html"/>
    <property name="dist.dir" value="dist"/>
    <property name="war.name" value="myproject.war"/>

    <!-- 컴파일 -->
    <target name="compile">
        <mkdir dir="${build.dir}"/>
        <javac srcdir="${src.dir}" destdir="${build.dir}" includeantruntime="false">
            <classpath>
                <fileset dir="${web.dir}/WEB-INF/lib">
                    <include name="**/*.jar"/>
                </fileset>
            </classpath>
        </javac>
    </target>

    <!-- WAR 생성 -->
    <target name="war" depends="compile">
        <mkdir dir="${dist.dir}"/>

        <!-- 클래스 파일 복사 -->
        <copy todir="${web.dir}/WEB-INF/classes">
            <fileset dir="${build.dir}"/>
        </copy>

        <!-- WAR 파일 생성 -->
        <war destfile="${dist.dir}/${war.name}" webxml="${web.dir}/WEB-INF/web.xml">
            <fileset dir="${web.dir}">
                <exclude name="WEB-INF/web.xml"/>
            </fileset>
        </war>

        <echo message="WAR 파일 생성 완료: ${dist.dir}/${war.name}"/>
    </target>

    <!-- 클린 -->
    <target name="clean">
        <delete dir="${build.dir}"/>
        <delete dir="${dist.dir}"/>
        <delete dir="${web.dir}/WEB-INF/classes"/>
    </target>
</project>
```

### web.xml 설정

**public_html/WEB-INF/web.xml:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                             http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <display-name>My Project</display-name>

    <!-- 세션 타임아웃 (30분) -->
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>

    <!-- 에러 페이지 -->
    <error-page>
        <error-code>404</error-code>
        <location>/error/404.jsp</location>
    </error-page>

    <error-page>
        <error-code>500</error-code>
        <location>/error/500.jsp</location>
    </error-page>

    <!-- 문자 인코딩 필터 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.apache.catalina.filters.SetCharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- 시작 페이지 -->
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

### Claude Code로 빌드

```
WAR 파일을 생성해줘.

1. ant clean으로 기존 빌드 정리
2. ant compile으로 Java 컴파일
3. ant war로 WAR 파일 생성
4. dist/myproject.war 파일 확인

빌드 오류가 있으면 수정해줘.
```

## Tomcat 배포

### 로컬 배포

**배포 디렉토리:**
```
C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\
```

**배포 방법:**

1. **WAR 파일 복사:**
```bash
copy dist\myproject.war "C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\"
```

2. **Tomcat 재시작:**
```bash
# Windows
catalina.bat stop
catalina.bat start

# Linux/Mac
catalina.sh stop
catalina.sh start
```

3. **접속 확인:**
```
http://localhost:8080/myproject/
```

### Claude Code로 배포

```
Tomcat에 배포해줘.

1. WAR 파일 생성 (ant war)
2. Tomcat webapps 폴더에 복사
3. Tomcat 재시작
4. http://localhost:8080/myproject/ 접속 확인

로그에 오류가 있으면 확인해줘.
```

## 원격 서버 배포

### SSH 배포

```bash
# WAR 파일 업로드
scp dist/myproject.war user@server:/tmp/

# SSH 접속
ssh user@server

# Tomcat에 배포
sudo cp /tmp/myproject.war /opt/tomcat/webapps/
sudo systemctl restart tomcat

# 로그 확인
tail -f /opt/tomcat/logs/catalina.out
```

### Claude Code로 원격 배포

```
원격 서버에 배포해줘.

서버 정보:
- 주소: myserver.com
- 사용자: deploy
- Tomcat 경로: /opt/tomcat

단계:
1. WAR 파일 생성
2. SCP로 업로드
3. SSH 접속 후 배포
4. Tomcat 재시작
5. 로그 확인

오류가 있으면 수정 방법 알려줘.
```

## 데이터베이스 마이그레이션

### 스키마 버전 관리

**schema_v1.sql:**

```sql
-- v1.0.0: 초기 스키마
CREATE TABLE tb_user (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(100) UNIQUE NOT NULL,
    passwd VARCHAR(64) NOT NULL,
    name VARCHAR(50) NOT NULL,
    reg_date VARCHAR(14) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE tb_board (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    view_count INT DEFAULT 0,
    reg_date VARCHAR(14) NOT NULL,
    FOREIGN KEY (user_id) REFERENCES tb_user(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**schema_v2.sql:**

```sql
-- v2.0.0: 댓글 기능 추가
CREATE TABLE tb_comment (
    id INT PRIMARY KEY AUTO_INCREMENT,
    board_id INT NOT NULL,
    user_id INT NOT NULL,
    content TEXT NOT NULL,
    reg_date VARCHAR(14) NOT NULL,
    FOREIGN KEY (board_id) REFERENCES tb_board(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES tb_user(id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 게시판에 댓글 수 컬럼 추가
ALTER TABLE tb_board ADD COLUMN comment_count INT DEFAULT 0;
```

### 마이그레이션 스크립트

**migrate.sh:**

```bash
#!/bin/bash

DB_HOST="localhost"
DB_USER="root"
DB_PASS="password"
DB_NAME="myproject"

echo "데이터베이스 마이그레이션 시작..."

# 현재 버전 확인
CURRENT_VERSION=$(mysql -h$DB_HOST -u$DB_USER -p$DB_PASS $DB_NAME -se "SELECT version FROM schema_version ORDER BY version DESC LIMIT 1")

if [ -z "$CURRENT_VERSION" ]; then
    CURRENT_VERSION="0"
fi

echo "현재 버전: v$CURRENT_VERSION"

# v2 마이그레이션
if [ "$CURRENT_VERSION" -lt "2" ]; then
    echo "v2 마이그레이션 실행..."
    mysql -h$DB_HOST -u$DB_USER -p$DB_PASS $DB_NAME < schema_v2.sql
    mysql -h$DB_HOST -u$DB_USER -p$DB_PASS $DB_NAME -e "INSERT INTO schema_version (version) VALUES (2)"
    echo "v2 마이그레이션 완료"
fi

echo "마이그레이션 완료!"
```

### Claude Code로 마이그레이션

```
DB 마이그레이션을 실행해줘.

현재 버전: v1.0.0
대상 버전: v2.0.0

변경 사항:
- tb_comment 테이블 추가
- tb_board에 comment_count 컬럼 추가

schema_v2.sql 실행 후 데이터 확인.
```

## 로그 관리

### 로그 설정

**log4j.properties:**

```properties
log4j.rootLogger=INFO, file, console

# 콘솔 출력
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n

# 파일 출력
log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.File=/var/log/myproject/app.log
log4j.appender.file.DatePattern='.'yyyy-MM-dd
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

### 로그 확인

```bash
# 실시간 로그 확인
tail -f /opt/tomcat/logs/catalina.out

# 오류 로그만 확인
grep ERROR /opt/tomcat/logs/catalina.out

# 최근 100줄
tail -n 100 /opt/tomcat/logs/catalina.out
```

### Claude Code로 로그 분석

```
Tomcat 로그를 분석해줘.

파일: /opt/tomcat/logs/catalina.out

분석 내용:
1. ERROR 발생 빈도
2. 자주 발생하는 오류 패턴
3. 성능 병목 지점
4. 개선 방안 제안

최근 1시간 로그 중심으로.
```

## 성능 최적화

### DB 인덱스 추가

```sql
-- 검색 성능 향상
CREATE INDEX idx_board_title ON tb_board(title);
CREATE INDEX idx_board_reg_date ON tb_board(reg_date);

-- JOIN 성능 향상
CREATE INDEX idx_board_user_id ON tb_board(user_id);
CREATE INDEX idx_comment_board_id ON tb_comment(board_id);

-- 복합 인덱스
CREATE INDEX idx_board_user_date ON tb_board(user_id, reg_date);
```

### 쿼리 최적화

**AS-IS (느림):**

```java
// N+1 문제
DataSet boards = boardDao.findAll("ORDER BY id DESC LIMIT 10");
while(boards.next()) {
    int userId = boards.i("user_id");
    DataSet user = userDao.findById(userId);  // 10번 쿼리
    user.next();
    boards.put("user_name", user.s("name"));
}
```

**TO-BE (빠름):**

```java
// JOIN 사용
String sql = "SELECT b.*, u.name as user_name " +
             "FROM tb_board b " +
             "LEFT JOIN tb_user u ON b.user_id = u.id " +
             "ORDER BY b.id DESC LIMIT 10";
DataSet boards = boardDao.executeQuery(sql);  // 1번 쿼리
```

### 커넥션 풀 설정

**context.xml:**

```xml
<Context>
    <Resource name="jdbc/MyDB"
              auth="Container"
              type="javax.sql.DataSource"
              maxTotal="100"
              maxIdle="30"
              maxWaitMillis="10000"
              username="root"
              password="password"
              driverClassName="com.mysql.cj.jdbc.Driver"
              url="jdbc:mysql://localhost:3306/myproject"/>
</Context>
```

### Claude Code로 최적화

```
게시판 목록 조회 성능을 최적화해줘.

현재 상황:
- 100개 게시글 조회 시 2초 소요
- 각 게시글마다 작성자 정보 조회 (N+1 문제)

개선 방법:
1. JOIN으로 한 번에 조회
2. 필요한 인덱스 추가
3. 쿼리 최적화

개선 후 성능 측정.
```

## 모니터링

### Health Check 페이지

**health.jsp:**

```jsp
<%@ page contentType="application/json; charset=utf-8" %><%@ include file="/init.jsp" %><%

// DB 연결 체크
boolean dbOk = false;
try {
    DB db = new DB();
    db.executeQuery("SELECT 1");
    dbOk = true;
} catch(Exception e) {
    dbOk = false;
}

// 디스크 공간 체크
File disk = new File("/");
long totalSpace = disk.getTotalSpace();
long freeSpace = disk.getFreeSpace();
double usagePercent = ((totalSpace - freeSpace) * 100.0) / totalSpace;

// 메모리 체크
Runtime runtime = Runtime.getRuntime();
long maxMemory = runtime.maxMemory();
long totalMemory = runtime.totalMemory();
long freeMemory = runtime.freeMemory();
long usedMemory = totalMemory - freeMemory;

j.put("status", dbOk ? "ok" : "error");
j.put("database", dbOk ? "connected" : "disconnected");
j.put("disk_usage_percent", Math.round(usagePercent * 100) / 100.0);
j.put("memory_used_mb", usedMemory / 1024 / 1024);
j.put("memory_max_mb", maxMemory / 1024 / 1024);
j.print();

%>
```

### 모니터링 스크립트

**monitor.sh:**

```bash
#!/bin/bash

URL="http://localhost:8080/myproject/health.jsp"

while true; do
    STATUS=$(curl -s $URL | jq -r '.status')

    if [ "$STATUS" != "ok" ]; then
        echo "[$(date)] 서비스 이상 감지!"
        # 알림 전송 (이메일, Slack 등)
    fi

    sleep 60  # 1분마다 체크
done
```

## 보안 설정

### HTTPS 설정

**server.xml:**

```xml
<Connector port="8443" protocol="HTTP/1.1"
           SSLEnabled="true"
           maxThreads="150"
           scheme="https"
           secure="true"
           clientAuth="false"
           sslProtocol="TLS">
    <SSLHostConfig>
        <Certificate certificateKeystoreFile="conf/keystore.jks"
                     certificateKeystorePassword="password"
                     type="RSA"/>
    </SSLHostConfig>
</Connector>
```

### 방화벽 설정

```bash
# 80, 443 포트만 허용
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 8080/tcp
sudo ufw enable
```

## 백업 및 복구

### DB 백업 스크립트

**backup.sh:**

```bash
#!/bin/bash

BACKUP_DIR="/backup/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="myproject"

# 백업
mysqldump -u root -p$DB_PASS $DB_NAME > $BACKUP_DIR/${DB_NAME}_$DATE.sql

# 압축
gzip $BACKUP_DIR/${DB_NAME}_$DATE.sql

# 7일 이상 된 백업 삭제
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete

echo "백업 완료: ${DB_NAME}_$DATE.sql.gz"
```

### 복구

```bash
# 압축 해제
gunzip /backup/mysql/myproject_20260129.sql.gz

# 복구
mysql -u root -p$DB_PASS myproject < /backup/mysql/myproject_20260129.sql
```

## 실전 프롬프트 예시

### 배포 자동화

```
배포 자동화 스크립트를 만들어줘.

작업 순서:
1. Git pull (최신 코드)
2. 테스트 실행 (ant test)
3. WAR 생성 (ant war)
4. Tomcat 중지
5. 기존 WAR 백업
6. 새 WAR 배포
7. Tomcat 시작
8. Health Check 확인

각 단계마다 오류 체크, 실패 시 롤백.
```

### 성능 진단

```
게시판 성능을 진단해줘.

체크 사항:
1. 느린 쿼리 찾기 (EXPLAIN 사용)
2. 인덱스 누락 확인
3. N+1 문제 확인
4. 커넥션 풀 상태

개선 방안 제시.
```

### 모니터링 대시보드

```
모니터링 대시보드 페이지를 만들어줘.

경로: /admin/dashboard.jsp

표시 내용:
- 서버 상태 (CPU, 메모리, 디스크)
- DB 연결 상태
- 최근 오류 로그 (10건)
- 트래픽 통계 (일별)
- 활성 세션 수

1분마다 자동 새로고침.
```

## 체크리스트

배포 전 확인사항:

- [ ] WAR 파일 정상 생성되는가?
- [ ] 프로덕션 DB 설정이 올바른가?
- [ ] 모든 테스트가 통과하는가?
- [ ] 로그 설정이 되어 있는가?
- [ ] 에러 페이지가 설정되어 있는가?
- [ ] HTTPS가 설정되어 있는가? (프로덕션)
- [ ] 방화벽이 설정되어 있는가?
- [ ] 백업 스크립트가 동작하는가?
- [ ] Health Check 페이지가 있는가?
- [ ] 모니터링이 설정되어 있는가?

## 관련 문서

- [프로젝트 시작하기](malgn-getting-started.md)
- [테스트 작성](malgn-testing.md)
- [보안 가이드라인](security.md)
- [코딩 규칙](coding-rules.md)
