### HATEOAS

---

    서버가 클라이언트에 하이퍼미디어를 통해 정보를 전달

HATEOAS 의존성 추가
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

```java
@SpringBootApplication
public class SpringtestdemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringtestdemoApplication.class, args);
    }
}

@RestController
public class SampleController {

    @GetMapping("/hello")
    public Resource hello() {
        Hello hello = new Hello();
        hello.setPrefix("Hello");
        hello.setName("Spring Boot");

        //Hateoas 생성자 파라미터로 T content 를 받는다
        Resource<Hello> resource = new Resource<>(hello);
        //linkTo : Link 인스턴스 생성
        //methodOn : Link 인스턴스에 SampleController.hello 를 참조할 수 있게 설정
        //withSelfRel : linkTo(methodOn()) 에서 생성한 참조 값을 Response 의 _links.self(withSelfRel) 에 설정
        //withRel("키값") : withSelfRel 은 self 키 값을 가지고 withRel 은 키 값을 설정하고 Link 값을 매핑한다
        resource.add(linkTo(methodOn(SampleController.class).hello()).withSelfRel());
        return resource;
    }
}

public class Hello {
    private String prefix;
    private String name;

    public String getPrefix() { return prefix; }
    public void setPrefix(String prefix) { this.prefix = prefix; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String toString(String prefix, String name) {
        return prefix + " " + name;
    }
}

@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void helloTest() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                //jsonPath 단위 테스트에서 사용하는 표현식
                .andExpect(jsonPath("$._links.self").exists());
    }
}
```

테스트 결과

```
           Status = 200
    Error message = null
          Headers = {Content-Type=[application/hal+json;charset=UTF-8]}
     Content type = application/hal+json;charset=UTF-8
             Body = {"prefix":"Hello","name":"Spring Boot","_links":{"self":{"href":"http://localhost/hello"}}}
    Forwarded URL = null
   Redirected URL = null
          Cookies = []
```