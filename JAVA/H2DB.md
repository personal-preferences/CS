<aside> 💡 H2 Database(이하 H2DB)는 Java로 작성된 오픈 소스 관계형 데이터베이스 관리 시스템(RDBMS)입니다. H2DB는 내장형 모드와 서버 모드를 지원하며, 빠르고 경량화된 데이터베이스로 개발 및 테스트 환경에서 주로 사용됩니다.

</aside>

# 특징

1. 내장형 모드와 서버 모드 지원
   - **내장형 모드**: 애플리케이션과 동일한 JVM 내에서 데이터베이스를 실행할 수 있어, 별도의 데이터베이스 서버 설치가 필요 없습니다. 개발 및 테스트 환경에서 유용합니다.
   - **서버 모드**: 독립된 서버로 동작하여 여러 클라이언트가 네트워크를 통해 동시에 접속할 수 있습니다.
2. 빠른 성능
   - H2DB는 매우 빠른 데이터베이스로 알려져 있으며, 빠른 응답 시간과 높은 처리 속도를 제공합니다.
3. 경량화
   - 데이터베이스 파일 크기가 작고, 메모리 사용량이 적어 경량화된 애플리케이션에 적합합니다.
4. SQL 호환성
   - 대부분의 SQL 표준을 지원하며, 다른 주요 데이터베이스 시스템(MySQL, PostgreSQL 등)과도 높은 호환성을 유지합니다.
5. Java 기반
   - 순수 Java로 작성되어 다양한 운영체제에서 실행 가능하며, Java 애플리케이션과의 통합이 용이합니다.
6. 다양한 저장 모드
   - 메모리 모드: 데이터가 메모리에만 저장되며, 애플리케이션 종료 시 데이터가 사라집니다.
   - 디스크 모드: 데이터가 파일 시스템에 저장되어 영구적으로 보관됩니다.
7. 웹 콘솔 제공
   - 내장 웹 서버를 통해 웹 브라우저에서 데이터베이스를 관리하고 쿼리를 실행할 수 있는 웹 콘솔을 제공합니다.

# 설정하기

### Gradle

```sql
plugins {
    id 'java'
}

group 'com.example'
version '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    testRuntimeOnly 'com.h2database:h2:2.1.214'
    testImplementation 'org.junit.jupiter:junit-jupiter-api:5.7.0'
    testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine:5.7.0'
}

test {
    useJUnitPlatform()
}
```

### YML

```sql
spring:
  datasource:
    url: jdbc:h2:mem:testdb          # H2 데이터베이스의 JDBC URL 설정, 메모리 모드 사용
    driver-class-name: org.h2.Driver # H2 드라이버 클래스 이름 설정
    username: sa                     # 데이터베이스 사용자 이름 설정
    password:                        # 데이터베이스 비밀번호 설정 (비워둠)
    initialization-mode: always      # 애플리케이션 시작 시 데이터베이스 초기화 모드 설정
  h2:
    console:
      enabled: true                  # H2 콘솔을 활성화
      path: /h2-console              # H2 콘솔에 접속할 경로 설정
```

### 실행 확인

애플리케이션을 실행하면 H2DB가 메모리 모드에서 실행되고, `application.yml` 파일에 설정한 대로 H2 콘솔을 통해 데이터베이스를 확인할 수 있습니다. 애플리케이션을 실행한 후 브라우저에서 `http://localhost:8080/h2-console`로 접속하여 설정한 데이터베이스를 확인할 수 있습니다.

![img](https://file.notion.so/f/f/73b72b54-ef99-4cf9-b18e-778678e237ad/33c66f14-c0e8-40c9-8525-dbfcc665c4e7/Untitled.png?id=a5dbed01-5556-4c35-b293-4723a91dd8ab&table=block&spaceId=73b72b54-ef99-4cf9-b18e-778678e237ad&expirationTimestamp=1722470400000&signature=2u7uxiEoHvux57f26NP1cTCIWP-WTZXMpAJ1LWPNnmQ&downloadName=Untitled.png)

# 테스트 코드에서 H2DB 사용하기

### 왜 mariaDB 두고 H2DB?

실제 사용중인 mariaDB를 사용했을 때의 문제점이 있습니다.

1. 테스트 코드를 실행하기 위해서는 항상 mariaDB와 연결되어 있어야한다.
2. 실제 DB를 사용하여 테스트 실행 시간이 오래걸린다.
3. 데이터베이스의 상태가 항상 동일하지 않다. (계속 사용중인 DB이므로 초기상태가 아니다)
4. 테스트 실행할 때마다 비용이 발생한다.

이러한 문제점들을 H2DB를 사용하게 되면 해소가 된다.

### mariaDB와 H2DB 두 가지를 동시에 사용하기

```
gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.mariadb.jdbc:mariadb-java-client'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testRuntimeOnly 'com.h2database:h2'
}
application.yml
spring:
  datasource:
    url: jdbc:mariadb://localhost:3306/your_database
    username: your_username
    password: your_password
    driver-class-name: org.mariadb.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MariaDBDialect
```

`application-test.yml` (테스트 환경 설정용 yml)

```sql
spring:
  datasource:
    url: jdbc:h2:mem:testdb;MODE=MariaDB   # MODE=MariaDB를 사용하여 H2가 MariaDB와 유사하게 동작
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.H2Dialect
  h2:
    console:
      enabled: true
테스트 코드
@SpringBootTest
@ActiveProfiles("test")
public class YourTestClass {
    // 테스트 코드
}
```