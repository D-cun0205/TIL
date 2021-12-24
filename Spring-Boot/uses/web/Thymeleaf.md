### Thymeleaf

---

    템플릿 엔진 : FreeMarker, Groovy, Thymeleaf, Mustache
    JSP 는 JAR 패키징에 동작하지 않고 WAR 패키징 해야한다
    Undertow 는 JSP 를 지원하지 않음

    Thymeleaf 참고 : https://thymeleaf.org/

```xml
<!-- thymeleaf 의존성 추가 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

```java
@SpringBootApplication
public class SpringtestdemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringtestdemoApplication.class, args);
    }
}

/* 핸들러 설정 */
@Controller
public class SimpleController {
    @GetMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("name", "sanghun");
        return "hello";
    }
}

/* 테스트 설정 */
@RunWith(SpringRunner.class)
@WebMvcTest(SimpleController.class)
public class SimpleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void webTest() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(view().name("hello"))
                .andExpect(model().attribute("name", is("sanghun")));
    }
}
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Spring Boot</title>
</head>
<body th:text="${name}">
</body>
</html>

<!-- 출력 결과 : sanghun -->
```