### 테스트

---

    @SpringBootTest 어노테이션은 @SpringBootApplication 어노테이션을 찾아가고
    이 어노테이션은 컴포넌트 스캔에 의해 어플리케이션 컨텍스트에 빈으로 해당되는 클래스를 
    등록하고 Test 클래스 안에 MockBean 과 매칭되는 빈이 있으면 교체해준다

* 통합 테스트
  * @AutoConfigureMockMvc
  * TestRestTemplate
  * WebTestClient
* 슬라이스 테스트
  * JsonTest
  * WebMvcTest
  * WebFluxTest
  * DataJpaTest ...


공통 디폴트 세팅

```java
@SpringBootApplication
public class SpringtestdemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringtestdemoApplication.class, args);
    }
}

@RestController
public class SimpleController {

    @Autowired
    SimpleService simpleService;

    @GetMapping("/hello")
    public String testString() {
        return "hello " + simpleService.getName();
    }
}

@Service
public class SimpleService {
    public String getName() {
        return "dcun";
    }
}
```

@AutoConfigureMockMvc

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class SimpleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void simpleTest() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk()) //200 ?
                .andExpect(content().string("hello dcun")) //요청에 대한 바디값과 같은지 ? 
                .andDo(print()); // request, response 과정 출력
    }
}
```

TestRestTemplate

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SimpleControllerTest {

    @Autowired
    TestRestTemplate testRestTemplate;

    @Test
    public void simpleTest() throws Exception {
        String result = testRestTemplate.getForObject("/hello", String.class); //내장톰켓 사용
        assertThat(result).isEqualTo("hello dcun"); //값 비교
    }
}
```

WebTestClient (Spring WebFlux 장점 : 비동기(많은 호출에 유리), 자동완성)

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class SimpleControllerTest {

    @Autowired
    WebTestClient webTestClient;

    @Test
    public void simpleTest() throws Exception {
        webTestClient.get().uri("/hello")
                .exchange()
                .expectStatus().isOk()
                .expectBody(String.class).isEqualTo("hello dcun");
    }
}
```

JsonTest

    @JsonTest 선언, JacksonTester<> 의존성 주입 후 Json 형태의 값 비교하여 사용

WebMvcTest

    웹 계층을 제외한 의존성은 모두 제거되며 필요시 @MockBean 으로 추가하여 테스트 진행

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SimpleController.class)
public class SimpleControllerTest {

    @MockBean
    SimpleService mockSimpleService; //mocking service

    @Autowired
    MockMvc mockMvc;

    @Test
    public void simpleTest() throws Exception {
        when(mockSimpleService.getName()).thenReturn("spring"); // 값 설정

        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello spring"))
                .andDo(print());
    }
}
```

### 테스트 유틸

OutputCapture

    Logging, System.out.print 출력을 캡쳐해주느 기능

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SimpleController.class)
public class SimpleControllerTest {

    @Rule
    public OutputCapture outputCapture = new OutputCapture();

    @MockBean
    SimpleService mockSimpleService;

    @Autowired
    MockMvc mockMvc;

    @Test
    public void simpleTest() throws Exception {
        when(mockSimpleService.getName()).thenReturn("spring");

        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello spring"))
                .andDo(print());

        assertThat(outputCapture.toString()).contains("holoman");
    }
}
```