### HTTP 메시지 컨버터

---

    @ResponseBody, @RequestBody 를 사용하면 HTTPMessageConverters 를 사용하여
    HTTP 요청 본문 < - > 자바 객체로 변환 해준다

핸들러, 매핑 클래스 설정

```java
@RestController
public class SimpleController {
    @PostMapping("/user/create")
    //@RestController 를 사용하면 리턴타입에 @ResponseBody 를 포함하고 있다
    public User create(@RequestBody User user) {
        return user;
    }
}

public class User {
    private String name;
    private int password;
    //getter, setter...
}
```

테스트 클래스

    JSON 테스트에서 요청, 응답에 대한 미디어 타입을 JSON
    XML 테스트에서 미디어타입을 요청(JSON), 응답(XML) 설정하고 검증코드를 수정

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SimpleController.class)
public class SimpleControllerTest {

    @Autowired
    MockMvc mockMvc;
    
    @Test //JSON 테스트
    public void createUser() throws Exception {
        String jsonString = "{\"name\":\"sanghun\",\"password\":\"123\"}";
        mockMvc.perform(post("/user/create")
                .contentType(MediaType.APPLICATION_JSON_UTF8)
                .accept(MediaType.APPLICATION_JSON_UTF8)
                .content(jsonString))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(equalTo("sanghun"))))
                .andExpect(jsonPath("$.password", is(equalTo(123))));
    }
    
    @Test //XML 테스트
    public void createUser() throws Exception {
        String jsonString = "{\"name\":\"sanghun\",\"password\":\"123\"}";
        mockMvc.perform(post("/user/create")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)
                        .accept(MediaType.APPLICATION_XML)
                        .content(jsonString))
                .andExpect(status().isOk())
                .andExpect(xpath("/User/name").string("sanghun"))
                .andExpect(xpath("/User/password").string("123"));
    }
}
```

    응답 미디어타입을 XML 로 설정하고 테스트할 때 사용되는 컨버터는 HttpMessageConvertersAutoConfiguration 가 import 하고 있는
    JacksonHttpMessageConvertersConfiguration 설정 빈의 MappingJackson2XmlHttpMessageConverterConfiguration 를
    사용하며 @ConditionalOnClass 로 XmlMapper.class 를 설정하고 있는데
    해당 클래스는 spring 에 내재되어 있지 않기 때문에 테스트가 정상적으로 작동하지않고 Exception 이 발생한다 
    그러므로 의존성을 아래와 같이 설정해야 사용 가능하다

```xml
<!-- 메시지 컨버터 추가 -->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
    <version>2.9.6</version>
</dependency>
```