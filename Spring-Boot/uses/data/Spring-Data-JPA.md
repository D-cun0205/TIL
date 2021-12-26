### 스프링 데이터 JPA

---

    스프링부트가 @EnableJpaRepositories 를 자동으로 등록해준다
    

Spring Data JPA, h2, postgresql 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

postgresql DB 정보

```properties
#SpringBoot 가 url의 postgresql 을 확인하고 driverClassName 을 유추하여 설정해준다
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

@Entity
public class Account {
    @Id @GeneratedValue
    private Long id;

    private String username;

    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Account account = (Account) o;
        return Objects.equals(id, account.id) && Objects.equals(username, account.username) && Objects.equals(password, account.password);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, username, password);
    }
}

public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUsername(String spring);
}
```

@DataJpaTest 를 사용하여 슬라이스 테스트를 진행한다  
슬라이스 테스트 시 인-메모리 데이터베이스를 사용 (H2)  
@SpringBootTest 를 사용하여 어플리케이션 전체 빈 등록하여 테스트도 가능하다

```java
@RunWith(SpringRunner.class)
//@SpringBootTest
@DataJpaTest
public class SpringDemoTest {

    @Autowired
    DataSource dataSource;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void di() throws SQLException {
        Account account = new Account();
        account.setUsername("spring");
        account.setPassword("1234");

        Account newAccount = accountRepository.save(account);
        assertThat(newAccount).isNotNull();

        Optional<Account> findUser = accountRepository.findByUsername("spring");
        assertThat(findUser).isNotEmpty();

        Optional<Account> nonFindUser = accountRepository.findByUsername("boot");
        assertThat(nonFindUser).isEmpty();
    }
}
```