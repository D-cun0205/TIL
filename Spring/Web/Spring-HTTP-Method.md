### Spring HTTP Method

---

* GET
* POST
* PUT
* PATCH
* DELETE

GET 요청

    클라이언트가 서버의 리소스를 요청
    캐싱 (조건적인 GET), if-modified-since 값으로 변경을 체크 및 성능 향상 
    브라우저 기록에 남는다
    북마크 기능
    GET 요청은 URL 에 정보가 보이므로 중요한 데이터를 보낼 때 사용하지 말자
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

POST 요청

    클라이언트가 서버의 리소스를 등록 및 수정
    POST 요청 본문에 데이터를 담아서 서버에 보낸다
    데이터 길이 제한이 없다

PUT 요청

    URI 에 해당하는 데이터를 새로 만들거나 수정
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

POST 와 PUT 차이점

    POST : 여러번의 신규 등록을 POST 요청으로 보내면 매번 새로운 등록을 진행한다
    PUT : 여러번의 신규 등록을 PUT 요청으로 보내면 처음에 새로운 등록(식별자를 통한) 이후 같은 등록(처음 생성한 식별자)을 반복한다

PATCH 요청

    기존 엔티티와 새 데이터의 차이점을 제외하고 PUT 과 동일
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

DELETE 요청

    URI 에 해당하는 리소스 삭제
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

매핑 사용 방법

```java
@Controller
//클래스 레벨의 설정으로 해당 클래스의 핸들러 메소드를 통일할 수 있다
@RequestMapping(method = RequestMethod.GET)
public class SampleController {

    @RequestMapping("/hello")
    @ResponseBody //응답 본문으로 뷰 대체
    public String hello() {
        return "hello";
    }

    //여러가지 메소드를 허용하고 싶은 경우 배열로 설정
    @RequestMapping(method = { RequestMethod.GET, RequestMethod.POST })
    public String arrMethodHandler() {
        return "";
    }
}
```


#### 패턴을 사용하는 매핑

    클래스에 @RequestMapping 선언 value 속성 값 입력시 클래스 value(매핑속성값) + 메소드 value(매핑속성값) 로 매핑을 시도한다
    요청에 매핑되는 핸들러가 여러개인 경우 가장 구체적인 핸들러에 매핑된다 

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @GetMapping("/?")
    @ResponseBody
    public String pattern1Test() {
        return "hello spring";
    }

    @GetMapping("/*")
    @ResponseBody
    public String pattern2Test() {
        return "hello spring";
    }

    @GetMapping("/**")
    @ResponseBody
    public String pattern3Test() {
        return "hello spring";
    }
}

@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    // ? 패턴을 사용하여 핸들러 매핑 (? 갯수 만큼의 글자)
    @Test
    public void pattern1Test() throws Exception {
        mockMvc.perform(get("/hello/spring1"))
                .andDo(print())
                .andExpect(content().string("hello spring"));
    }

    // * 패턴을 사용하여 핸들러 매핑 (패스를 추가 하지않은 경우 여러 글자)
    @Test
    public void pattern2Test() throws Exception {
        mockMvc.perform(get("/hello/springabcd"))
                .andDo(print())
                .andExpect(content().string("hello spring"));
    }

    // * 패턴을 사용하여 핸들러 매핑 (패스 개수 상관없이 여러글자)
    @Test
    public void pattern3Test() throws Exception {
        mockMvc.perform(get("/hello/springabcd/wegwegw/aertnaert"))
                .andDo(print())
                .andExpect(content().string("hello spring"));
    }
}
```


#### 정규 표현식 매핑

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @GetMapping("/{name : [a-z]+}")
    @ResponseBody
    public String pattern1Test(@PathVariable String name) {
        return "hello " + name;
    }
}
```


#### URI 확장자 매핑 지원

    아래와 같은 핸들러가 있는 경우 스프링 프레임워크는 확장자를 제거한 요청을 해도 자동으로 매핑 해준다
    스프링 부트 또는 스프링 4.2 버전 이상부터는 RFD Attack 으로 인해 기본 사용조건이 false가 되었고
    그러므로 필요하면 아래 조건과 같이 받을 수 있는 핸들러를 만들면 되지만 권장하지 않는다

```java
@Controller
@RequestMapping("/hello")
public class SampleController {

    @GetMapping("/spring.json")
    @GetMapping("/spring.zip")
    @GetMapping("/spring.xml")
    @ResponseBody
    public String pattern1Test(@PathVariable String name) {
        return "hello " + name;
    }
}
```


#### 미디어 타입 매핑하기

    클래스와 메소드의 content-type, accept 를 지정하면 두개의 컨텐츠 타입을 사용하는게 아닌
    오버라이드 되어 메소드에서 설정한 타입들을 사용하게 된다

```java
@Controller
@RequestMapping(
        value = "/sample",
        consumes = MediaType.APPLICATION_XML_VALUE,
        produces = MediaType.APPLICATION_JSON_VALUE
)
public class SampleController {
    @RequestMapping(
            value = "/hello",
            consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.TEXT_PLAIN_VALUE
    )
    @ResponseBody
    public String test() {
        return "hello";
    }
}
```

미디어 타입 매핑 테스트

    요청 : 핸들러에 consumes 속성 값이 있는 경우 테스트에서도 동일한 타입을 지정해야 Unsupported Media Type 이 발생하지 않는다
    응답 : 핸들러에 produces 속성 값이 있고 받는쪽에서 속성값을 주지 않아도 모든 타입을 수용하기 때문에 에러가 발생하지 않는다
          응답의 경우는 양쪽의 속성 값이 다른경우 Not Acceptable 이 발생한다

```java
@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void test() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("hello"));
    }
}
```


#### 헤더, 매개변수

    @RequestMapping 의 속성 값 params, headers 로 매핑을 제한할 수 있다
    문자열에 "!" 값을 넣고 false 조건을 만들어서 테스트할 수 있다 

```java
public class SampleController {

    // params, headers 값을 사용하여 핸들러를 매핑
    @RequestMapping(value = "/param", params = "name=spring")
    @ResponseBody
    public String paramTest() {
        return "param";
    }

    @RequestMapping(value = "/header", headers = HttpHeaders.FROM)
    @ResponseBody
    public String headerMappingTest() {
        return "header";
    }

    @RequestMapping(value = "/author", headers = HttpHeaders.AUTHORIZATION + "=123")
    @ResponseBody
    public String headerAuthorTest() {
        return "author";
    }
}
```

헤더, 매개변수 값을 사용한 테스트

```java
@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void paramTest() throws Exception {
        mockMvc.perform(get("/param")
                        .param("name", "spring"))
                .andDo(print())
                .andExpect(status().isOk());
    }
    
    @Test
    public void header() throws Exception {
        mockMvc.perform(get("/header")
                        .header(HttpHeaders.FROM, "localhost"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(content().string("header"));
    }

    @Test
    public void author() throws Exception {
        mockMvc.perform(get("/author")
                        .header(HttpHeaders.AUTHORIZATION, "123"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```


#### 커스텀 어노테이션

메타(Meta) 어노테이션

    어노테이션에 사용할 수 있는 어노테이션
    스프링이 제공하는 대부분의 어노테이션은 메타 어노테이션 사용 가능

조합(Composed) 어노테이션

    한개 혹은 여러 메타 어노테이션을 조합해서 만든 어노테이션
    코드를 간결하게 줄일 수 있고 구체적인 의미를 부여할 수 있다

```java
@Documented //해당 어노테이션을 사용한 코드의 문서에 어노테이션에 대한 정보를 표기
//어노테이션 적용 대상
@Target(
        //{ElementType.METHOD, ElementType.FIELD} //배열로 여러개를 지정
        ElementType.METHOD
)
//어노테이션 보유 기간 설정
@Retention(
        //RetentionPolicy.CLASS //클래스가 생성되고 어노테이션이 제거
        RetentionPolicy.RUNTIME //런타임시에도 작동
        //RetentionPolicy.SOURCE //주석
)
@RequestMapping(method = RequestMethod.GET, value = "/hello")
public @interface GetHelloMapping {
}

@Controller
public class SampleController {

    @GetHelloMapping
    @ResponseBody
    public String test() {
        return "hello";
    }
}

@WebMvcTest(SampleController.class)
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void test() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```