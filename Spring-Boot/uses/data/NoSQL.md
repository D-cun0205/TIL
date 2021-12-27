### NoSQL

---

* redis
* mongoDB
* Neo4j


#### Redis

    호출 방법
    StringRedisTemplate 를 사용하여 등록했을 때
    get : key 값 ex) get name 입력, value 값 리턴
    오브젝트를 등록했을 때
    hget : hash field + filed, ex) hget 오브젝트해시값 name, value 값 리턴  
    hgetall : hash field, ex) hgetall 오브젝트해시값, 오브젝트가 가지고 있는 모든 값 리턴

redis docker image

```
docker run -p 6379:6379 --name redis_boot -d redis
```

redis 의존성

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

redis 속성 설정

```properties
redis doc 참조..
```

```java
@SpringBootApplication
public class SpringDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDemoApplication.class, args);
    }
}


@RedisHash
public class Account {
    @Id
    private String id;
    private String name;
    private String email;

    //getter, setter...
}


public interface AccountRepository extends CrudRepository<Account, String> {}


@Component
public class RedisAppRunner implements ApplicationRunner {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> value = redisTemplate.opsForValue();
        value.set("name", "sanghun");
        value.set("spring", "boot");
        value.set("cafe", "hollys");

        Account account = new Account();
        account.setName("dcun");
        account.setEmail("dcun@email.com");
        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getEmail());
        System.out.println(byId.get().getName());
    }
}
```

#### MongoDB

    mongodb 명령어
    db : db list
    use : use dbname (db 접속)
    db.'collections name'.find({}) : 데이터 출력
    의존성만 추가, 어노테이션 사용 시 운영, 테스트 DB에 대한 별도의 설정이 필요 없다

mongodb docker image

```
docker run -p 27017:27017 --name mongo_boot -d mongo
```

mongDB, mongDB embedded 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
<dependency>
    <groupId>de.flapdoodle.embed</groupId>
    <artifactId>de.flapdoodle.embed.mongo</artifactId>
</dependency>
```

```java
@Document(collection = "accounts")
public class Account {
    @Id
    private String id;
    private String name;
    private String email;

    //getter, setter...
}


public interface AccountRepository extends MongoRepository<Account, String> {
    //쿼리메소드에 의해 쿼리 자동생성
    Optional<Account> findByEmail(String email);
}


//ApplicationRunner 를 @Bean 으로 사용(람다표현식)
@SpringBootApplication
public class SpringDemoApplication {

    @Autowired
    MongoTemplate mongoTemplate;

    @Autowired
    AccountRepository accountRepository;

    public static void main(String[] args) {
        SpringApplication.run(SpringDemoApplication.class, args);
    }

    @Bean
    public ApplicationRunner applicationRunner() {
        return args -> {
            Account account = new Account();
            account.setName("spring");
            account.setEmail("spring@spring.co.kr");
            mongoTemplate.insert(account);

            Account account2 = new Account();
            account2.setName("mongo");
            account2.setEmail("mongo@mongo.co.kr");
            accountRepository.save(account2);

            Optional<Account> byId = accountRepository.findById(account2.getId());
            System.out.println(byId.get().getName());
            System.out.println(byId.get().getEmail());
        };
    }
}
```

TEST

```java
@RunWith(SpringRunner.class)
@DataMongoTest //mongo 슬라이스 테스트
public class SpringDemoApplicationTest {

    @Autowired
    AccountRepository accountRepository;

    @Test
    public void mongoDBTest() {
        Account account = new Account();
        account.setName("mongo");
        account.setEmail("mongo@email.co.kr");
        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        assertThat(byId).isNotEmpty();

        Optional<Account> byEmail = accountRepository.findByEmail(account.getEmail());
        assertThat(byEmail).isNotEmpty();
    }
}
```

#### Leo4j

```
docker run -p 7474:7474 -p 7687:7687 --name neo4j_boot -d neo4j
```

```properties
spring.data.neo4j.uri=bolt://localhost:7687
spring.data.neo4j.username=neo4j
spring.data.neo4j.password=1111
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```

```java
@NodeEntity
public class Account {
    @Id @GeneratedValue
    private Long id;
    private String username;
    private String userEmail;

    @Relationship
    private Set<Role> roleSet = new HashSet<>();

    //getter, setter...
}

@NodeEntity
public class Role {

    @Id @GeneratedValue
    private Long id;
    private String grade;

    //getter, setter...
}

@Component
public class Neo4jAppliationRunner implements ApplicationRunner {

    @Autowired
    SessionFactory sessionFactory;

    @Autowired
    AccountRepository accountRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setUsername("neo4j");
        account.setUserEmail("neo4j@email.co.kr");

        Role role = new Role();
        role.setGrade("admin");

        account.getRoleSet().add(role);

        accountRepository.save(account);

        Optional<Account> byId = accountRepository.findById(account.getId());
        System.out.println(byId.get().getUsername());
        System.out.println(byId.get().getUserEmail());

        Session session = sessionFactory.openSession();
        session.save(account);
        sessionFactory.close();

        System.out.println("finished");
    }
}
```