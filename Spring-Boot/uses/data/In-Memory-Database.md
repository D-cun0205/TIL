### H2 

---

    ApplicationRunner 를 구현하여 인메모리 데이터베이스(H2) 사용하는 방법
    H2 console
        Saved Settings : Generic H2 (Embedded)
        Saved Settings 를 변경하면 driver class, url, name 값은 자동으로 설정된다 

H2, JDBC 의존성 추가

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>
```

H2 console 사용

```properties
spring.h2.console.enabled=true
```

```java
@SpringBootApplication
public class SpringDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDemoApplication.class, args);
    }
}

@Configuration
public class H2AppRunner implements ApplicationRunner {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try(
                Connection connection = dataSource.getConnection()
        ) {
            String sql = "CREATE TABLE USER (ID INTEGER NOT NULL , NAME VARCHAR(255) , PRIMARY KEY(ID))";
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            preparedStatement.executeUpdate();
        }
        jdbcTemplate.execute("INSERT INTO USER VALUES (1, 'spring')");
    }
}
```