### Spring HTTP Method

---

* GET
* POST
* PUT
* PATCH
* DELETE

#### GET 요청

    클라이언트가 서버의 리소스를 요청
    캐싱 (조건적인 GET), if-modified-since 값으로 변경을 체크 및 성능 향상 
    브라우저 기록에 남는다
    북마크 기능
    GET 요청은 URL 에 정보가 보이므로 중요한 데이터를 보낼 때 사용하지 말자
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

#### POST 요청

    클라이언트가 서버의 리소스를 등록 및 수정
    POST 요청 본문에 데이터를 담아서 서버에 보낸다
    데이터 길이 제한이 없다

#### PUT 요청

    URI 에 해당하는 데이터를 새로 만들거나 수정
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

POST 와 PUT 차이점

    POST : 여러번의 신규 등록을 POST 요청으로 보내면 매번 새로운 등록을 진행한다
    PUT : 여러번의 신규 등록을 PUT 요청으로 보내면 처음에 새로운 등록(식별자를 통한) 이후 같은 등록(처음 생성한 식별자)을 반복한다

#### PATCH 요청

    기존 엔티티와 새 데이터의 차이점을 제외하고 PUT 과 동일
    idemponent (멱등 : 요청을 재시도 해도 기존 요청과 동일하게 작동)

#### DELETE 요청

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