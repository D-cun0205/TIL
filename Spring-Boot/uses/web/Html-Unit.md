### Html Unit

---

Html 단일 테스트 참고 : https://htmlunit.sourceforge.io/

```xml
<!-- Html 단일 테스트 의존성 추가 -->
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>htmlunit-driver</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>net.sourceforge.htmlunit</groupId>
    <artifactId>htmlunit</artifactId>
    <scope>test</scope>
</dependency>
```

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Spring Boot</title>
</head>
<body>
<h1 th:text="${name}"></h1>
</body>
</html>
```

```java
@RunWith(SpringRunner.class)
@WebMvcTest(SimpleController.class)
public class SimpleControllerTest {

    @Autowired
    WebClient webClient;

    @Autowired
    MockMvc mockMvc;

    @Test
    public void webTest() throws Exception {
        HtmlPage page = webClient.getPage("/hello");
        HtmlHeading1 h1 = page.getFirstByXPath("//h1");
        assertThat(h1.getTextContent()).isEqualToIgnoringCase("sanghun");
    }
}
```