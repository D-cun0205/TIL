### Formatter 

---

spring boot web, test 의존성 추가

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

response 의 값을 객체에 매핑하고 싶을 때 Formatter 를 적용

```java
public class Person {
    private String name;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

@RestController
public class SampleController {

    @GetMapping("/hello")
    public String hello(@RequestParam("name") Person person) {
        return "Hello World " + person.getName();
    }
}
```

Spring Framework

    Formatter 구현체를 자바 Config 에 addFormatter 로 추가

```java
public class PersonFormatter implements Formatter<Person> {

    @Override
    public Person parse(String s, Locale locale) throws ParseException {
        Person person = new Person();
        person.setName(s);
        return person;
    }

    @Override
    public String print(Person person, Locale locale) {
        return person.getName();
    }
}

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addFormatter(new PersonFormatter());
    }
}
```

Spring Boot

    Formatter 구현체를 빈으로 등록해주면 Spring Boot가 대신 addFormatter 로 등록해준다

```java
@Component
public class PersonFormatter implements Formatter<Person> {

    @Override
    public Person parse(String s, Locale locale) throws ParseException {
        Person person = new Person();
        person.setName(s);
        return person;
    }

    @Override
    public String print(Person person, Locale locale) {
        return person.getName();
    }
}
```

TEST

    Spring Boot 기준 Formatter 를 빈으로 등록하고 @WebMvcTest 를 진행하면
    error 가 발생하는데 이유는 WebMvcTest 는 슬라이스 테스트로 Web 에 관련된 빈만 등록
    그러므로 Formatter에 대한 빈을 추가하지 않으므로 등록된 Formatter가 없어서 에러가 발생
    해결방법으로 @AutoConfigureMockMvc 를 추가하여 Formatter를 빈으로 자동주입 받거나
    @SpringBootTest 를 사용하여 전체를 빈으로 등록하고 테스트하여 해결 가능하다

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
//@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        this.mockMvc.perform(get("/hello").param("name", "sanghun"))
                .andDo(print())
                .andExpect(content().string("Hello World sanghun"));
    }
}
```