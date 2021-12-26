### 관계형 데이터베이스 관리 시스템

---

    Mysql, MariaDB 의 설정은 같다 
    도커 컨테이너에 이미지를 실행할 때 이미지로 mysql, mariadb 를 구분한다

docker mysql + mariadb

```
docker run -p 3306:3306 --name mysql_boot -e MySQL_ROOT_PASSWORD=1
                -e MYSQL_DATABASE=springboot -e MYSQL_USER=spring
                -e MYSQL_PASSWORD=password -d mysql

mysql -u -root -p
```

docker postgres

```
docker run -p 5432:5432 -e POSTGRES_PASSWORD=password -e POSTGRES_USER=spring
                        -e POSTGRES_DB=springboot --name postgres_boot -d postgres
                        
su - postgres (role error)

psql springboot (role error) 

psql -U spring springboot (role error 해결 방법) 
```

Mysql + MariaDB, Postgres

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

```properties
spring.datasource.hikari.maximum-pool-size=4

#spring.datasource.url=jdbc:mysql://localhost:3306/springboot
spring.datasource.url=jdbc:postgresql://localhost:5432/springboot
spring.datasource.username=sanghun
spring.datasource.password=pass
```

```java
@SpringBootApplication
public class SpringDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDemoApplication.class, args);
    }
}

@Configuration
public class MySQLAppRunner implements ApplicationRunner {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        try(
                Connection connection = dataSource.getConnection()
        ) {
            //postgresql 에서 user 를 키워드로 사용하고 있어서 user 테이블 생성 불가
            String sql = "CREATE TABLE ACCOUNT (ID INTEGER NOT NULL , NAME VARCHAR(255) , PRIMARY KEY(ID))";
            PreparedStatement preparedStatement = connection.prepareStatement(sql);
            preparedStatement.executeUpdate();
        }
        jdbcTemplate.execute("INSERT INTO ACCOUNT VALUES (1, 'spring')");
    }
}
```