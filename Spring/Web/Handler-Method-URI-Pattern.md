### 핸들러 메소드 URI 패턴

---

@PathVariable

    요청 URI 패턴의 일부를 핸들러 메소드 아규먼트로 받는 방법 ex) /{전달받을 키값}
    타입 자동 변환 지원
    디폴트로 값이 반드시 있어야 하며 속성값에 boolean 값으로 변경 가능
    Optional 지원 ex) (@PathVariable Optioanl<Integer> id)

```java
@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void test() throws Exception {
        mockMvc.perform(get("/handlerMethodTest/1"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("id").value("1"));
    }
}

@Controller
public class SampleController {

    @GetMapping("/handlerMethodTest/{id}")
    @ResponseBody
    public Events test(@PathVariable Integer id) {
        Events events = new Events();
        events.setId(id);
        return events;
    }
}
```

@MatrixVariable

    요청 URI 패턴에서 키/값 쌍의 데이터를 메소드 아규먼트로 받는 방법
    타입 변환, 값 필수 여부, Optional 은 PathVariable 과 동일
    해당 기능은 비활성화 되어 있으며 사용하기 위해서 설정이 필요하다

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configurePathMatch(PathMatchConfigurer configurer) {
        UrlPathHelper urlPathHelper = new UrlPathHelper();
        urlPathHelper.setRemoveSemicolonContent(false);
        configurer.setUrlPathHelper(urlPathHelper);
    }
}
```

테스트 1, 2 번 다른점은 @PathVairable 사용 후 '/' 있고 없고

```java
@Controller
public class SampleController {

    @GetMapping("/matrixArgsTest1/{id}{name}")
    @ResponseBody
    public Events test1(@PathVariable Integer id, @MatrixVariable String name) {
        Events events = new Events();
        events.setId(id);
        events.setName(name);
        return events;
    }

    @GetMapping("/matrixArgsTest2/{id}/{name}")
    @ResponseBody
    public Events test2(@PathVariable Integer id, @MatrixVariable String name) {
        Events events = new Events();
        events.setId(id);
        events.setName(name);
        return events;
    }
}

@RunWith(SpringRunner.class)
@WebMvcTest
public class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void test1() throws Exception {
        mockMvc.perform(get("/matrixArgsTest1/1;name=spring"))
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void test2() throws Exception {
        mockMvc.perform(get("/matrixArgsTest2/1/name=spring"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```