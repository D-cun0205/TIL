### @MVC 메소드 파라미터 종류

* HttpServletRequest, HttpServletResponse
* HttpSession
* WebRequest, NativeWebRequest
* Locale
* InputStream, Reader
* OutputStreamm Writer
* @PathVariable
* @RequestParam
* @CookieValue
* @RequestHeader
* Map, Model, ModelMap
* @ModelAttribute
* Errors, bindingResult
* SessionStatus
* @RequestBody

HttpServletRequest, HttpServletResponse

    서블릿 오브젝트

HttpSession

    HttpServletRequest 를 통해서 가져올 수 있고 HTTP 세션만 필요한 경우 직접 선언하여 받는다

WebRequest, NativeWebRequest

    서블릿 API 에 종속적이지 않은 오브젝트 타입이며 HttpServletRequest 요청정보를 가지고 있다
    NativeWebRequest 는 WebRequest 내부의 환경종속적인 오브젝트를 가져올 수 있다

Locale

    DispatcherServlet 의 지역정보 리졸버가 결정한 Locale 오브젝트를 받는다

InputStream, Reader

    HttpServletRequest 의 getInputStream() 을 통해 받을 수 있는 콘텐츠 스트림, Reader 타입 오브젝트 제공

OutputStreamm Writer

    HttpServletResponse 의 getOutputStream() 으로 가져올 수 있는 출력용 콘텐츠 스트림, Writer 타입 오브젝트 제공

@PathVariable

    @RequestMapping URL 에 {} 로 들어가는 패스 변수를 받는다
    기존에 사용하는 쿼리스트링 방식을 URL 패스로 풀어서 쓰는 방식으로 사용
    ex) 쿼리스트링 : /user/view?id=10, URL 패스 : /user/view/10

```java
//URL 패스를 사용하여 파라미터를 받고 사용하는 방법
//"/user/{id}/view/{password}" 처럼 다중 패스 변수도 이용할 수 있다
//타입에 맞지 않은 파라미터가 들어오고 예외처리를 해주지 않는다면 400 - Bad Request 응답 코드 전달
@Controller
public class UserController {
    @RequestMapping("/user/view/{id}")
    public String view(@PathVariable("id") int id) { return ""; }
}
```

@RequestParam

    단일 HTTP 요청 파라미터를 메소드 파라미터에 넣어주는 어노테이션
    가져올 요청 파라미터의 이름을 @RequstParam 어노테이션의 기본 값으로 지정
    요청 파라미터의 값은 메소드 파라미터의 타입에 따라 적절하게 변환된다
    required 와 defaultValue 속성을 설정할 수 있다

```java
@Controller
public class UserController {
    //@RequestParam 을 여러개 사용할 수 있다
    @RequestMapping("/user/view")
    public String view(@RequestParam("id") int id) { return ""; }

    //@RequestParam 에 파라미터 이름을 지정하지 않고 Map 타입으로 선언하여 모든 요청 파라미터를 담은 맵으로 받을 수 있다
    @RequestMapping("/user/view")
    public String view(@RequestParam Map<String, String> params) { return ""; }
}
```

@CookieValue

    HTTP 요청과 함께 전달된 쿠키 값을 메소드 파라미터에 넣어주고 사용할 수 있다
    어노테이션의 기본 값에 쿠키의 이름을 지정해주면 된다
    메소드 파라미터 이름과 쿠키 값이 동일하다면 쿠키 이름은 생략 가능
    required 와 defaultValue 속성을 설정할 수 있다

```java
@Controller
public class UserController {
    @RequestMapping("/user/view")
    public String check(@CookieValue(value="auth") int id) { return ""; }
}
```

@RequestHeader

    HTTP 요청 헤더정보를 메소드 파라미터에 넣어주는 어노테이션

```java
@Controller
public class UserController {
    @RequestMapping("/user/view")
    public String check(@RequestHeader("Host") String host) { return ""; }
}
```

Map, Model, ModelMap

    모델정보를 담는 데 사용할 수 있는 오브젝트 전달
    핸들러 어댑터에서 미리 만들어 제공해주는 것을 사용하여 파라미터에 선언하여 사용하면
    별도의 return 없이 DispatcherServlet 에 전달할 수 있다

@ModelAttribute

    메소드 레벨에 적용과 메소드 파라미터에 적용하는 두가지 방법이 있으며 사용 목적이 다르니 주의
    예를들어 웹 페이지의 사용자 폼 데이터를 받는 경우 받을 수 있는 파라미터는 다양하다
    사용자 명, 패스워드, 주소, 핸드폰 번호, 이메일 등등 그리고 파라미터를 받는 메소드를 정의 하면 아래와 같다

```java
@Controller
public class UserController {
    @RequestMapping("/user/view")
    public String check(
            @RequestParam("name") String name,
            @RequestParam("password") String password,
            @RequestParam("address") String address,
            @RequestParam("phone") String phone,
            @RequestParam("email") String email) { 
        return "";
    }
}
```

    문제는 폼의 데이터가 추가되면 추가될수록 소스코드는 길어지고
    서비스 또는 데이터 엑세스 단계로 넘겨야 할 경우 파라미터를 특정 VO 에 매핑하거나
    선언되어 있는 @RequestParam 개수만큼 한개 한개 수동으로 넘겨야 하는데
    이러한 문제점을 해결하기 위해서 @ModelAttribute 를 사용하여 깔끔하게 해결할 수 있다

```java
@Controller
public class UserController {
    @RequestMapping("/user/view")
    public String check(@ModelAttribute User user) { 
        return "";
    }
}
```  

    요청과 매핑하기 위한 User 클래스에 @RequestParam 으로 넘어올 필드를 선언해야 한다

Errors, bindingResult

    @RequestParam 의 경우 HTTP 요청에 의한 파라미터를 타입 체크하고 타입이 맞지 않은 경우 에러가 발생한다
    그러나 @ModelAttribute 는 에러라고 판단하지 않고 검증 작업의 한가지 결과정도로 생각하고 넘어간다
    이럴 때 사용할 수 있는 방법이며 사용할 때는 @ModelAttribute 뒤에 선언해야
    앞에 선언된 파라미터에 대한 검증 작업 발생 오류를 전달해준다

SessionStatus

    Model 오브젝트를 세션에 저장했다가 다음 페이지에서 다시 활용할 수 있도록 해주는 기능이 있는데
    더이상 세션 내에 모델 오브젝트가 필요 없는 경우 코드에서 작업 완료 메소드를 호출하여
    저장된 오브젝트를 제거해줘야 하는데 그 역할을 하는게 SessionStatus 이다

@RequestBody

    이 어노테이션을 사용하게되면 HTTP 요청의 본문이 그대로 전달된다
    XML, JSON 기반의 메시지를 사용하는 요청에 유용
    XML 본문을 가지고 있는 요청은 MarshallingHttpMessageConverter 를 이용하고
    JSON 타입의 메시지는 MappingJacksonHttpMessageConverter를 이용하여
    파라미터 타입으로 변환하여 전달 해준다

@Value

    시스템 프로퍼티, 다른 빈의 프로퍼티 값, SpEL을 이용한 상수, 특정 메소드를 호출한 값, 조건식에 사용

```java
public class Test {
    //시스템 프로퍼티를 가져오는 예시
    public String hello(@Value("#{systemProperties['os.name']}") String osName){}
}
```

@Valid

    보통 @ModelAttribute 와 같이 사용하며 모델 오브젝트의 검증 방법을 지정하는데 사용