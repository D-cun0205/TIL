### Spring Web skill, MVC

---

#### 웹 프레젠테이션 계층 : 사용자 또는 클라이언트 시스템과 연동하는 책임

* 스프링 웹 프레임 워크
  * 스프링 서블릿/스프링 MVC
  * 스프링 포틀릿  (작은 단위의 프레젠테이션 컴포넌트)


    스프링 서블릿/스프링 MVC 는 스프링이 직접 제공하는 서블릿 기반의 MVC 프레임워크다
    프론트 컨트롤러 역할의 핵심 엔진으로 DispatcherServlet를 사용하며 다양한 종류의 컨트롤러를 동시에 사용 가능

#### DispatcherServlet과 MVC 아키텍처

DispatcherServlet : 스프링 MVC 가 프론트 컨트롤러로 사용하는 서블릿

* 작업 흐름
  * DispatcherServlet의 HTTP 요청 접수
  * DispatcherServlet에서 컨트롤러로 HTTP 요청 위임
  * 컨트롤러의 모델 생성과 정보 등록
  * 컨트롤러의 결과 리턴 : 모델과 뷰
  * DispatcherServlet의 뷰 호출
  * 모델 참조
  * HTTP 응답 전달


흐름 1.

    Java Server Servlet Container는 HTTP 요청에 대한 설정으로 Spring DispatcherServlet이 할당 되어 있으면 요청을 전달

```xml
<!-- 아래의 서블릿 매핑은 URL이 /app 으로 시작하는 모든 요청을 DispatcherServlet에 전달, 특정 확장자만 매핑하는 방법도 가능 -->
<servlet-mapping>
  <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
  <url-pattern>/app/*</url-pattern>
</servlet-mapping>
```

    DispatcherServlet은 전처리 작업이 설정되어있으면 먼저 수행한다 보통 보안, 파라미터조작, 한글 디코딩...

흐름 2.

    DispatcherServlet은 URL, 파라미터 정보, HTTP 명령 정보로 어떤 컨트롤러에 작업을 위임할지 결정
    컨트롤러 선정은 DispatcherServlet의 핸들러 매핑 전략 이용
      * 핸들러 매핑 전략 : 사용자 요청을 기준으로 어떤 핸들러(컨트롤러)에게 작업을 위임할지 결정
    DispatcherServlet이 요청을 위임할 오브젝트를 찾아서 전달하기위해 메소드를 실행시키는 방법으로 어댑터 이용
      * 어댑터(오브젝트) 패턴 : 특정 클래스의 인터페이스를 클라이언트가 기대하는 다른 인터페이스 변환
    DispatcherServlet이 핸들러 어댑터에 HttpServletRequest 타입 오브젝트 전달
    핸들러 어댑터는 HttpServletRequest를 컨트롤러에서 받을 수 있는 파라미터로 변환해서 전달

흐름 3.

    사용자 요청 해석, 비즈니스 로직 수행을 위해 서비스 계층 오브젝트에 작업 위임, 결과 받아서 모델 생성
      * 모델 : 이름 , 오브젝트의 값

흐름 4.

    뷰도 결국엔 하나의 오브젝트이며 컨트롤러가 뷰 오브젝트를 직접 리턴 가능하나 사용하지 않음
    뷰의 논리적인 이름을 리턴하면 DispatcherServlet의 전략인 뷰 리졸버가 뷰 오브젝트를 생성
    최종적으로 ModelAndView에 모델, 뷰를 등록하여 어댑터를 통해 DispatcherServlet에 전달

흐름 5. 흐름 6.

    뷰 오브젝트에게 모델을 전달하고 클라이언트에게 전달할 최종 결과물 생성 요청 (HTML 생성)
    뷰가 생성되면 HttpServletResponse 오브젝트에 Set

흐름 7.

    HttpServletResponse를 돌려줄 때 후 처리기가 있으면 후속 작업을 진행하고 최종 결과를 서블릿 컨테이너에 전달
    서블릿 컨테이너는 HttpServletResponse에 담긴 정보를 HTTP 응답으로 만들어 클라이언트에게 전송하고 작업 종료

#### DispatcherServlet의 DI 가능한 전략

* DI 전략 종류
  * HandlerMapping
  * HandlerAdapter
  * HandlerExceptionResolver
  * ViewResolver
  * LocalResolver
  * ThemeResolver
  * RequestToViewNameTranslator

HandlerMapping

    URL, 요청 정보를 기준으로 어떤 핸들러 오브젝트, 즉 사용할 컨트롤러 결정
    DispatcherServlet은 하나 이상의 핸들러를 가질 수 있다
    디폴트로 BeanNameUrlHandlerMapping, DefaultAnnotationHandlerMapping 설정

HandlerAdapter

    핸들러 매핑으로 선택한 핸들러(컨트롤러)를 DispatcherServlet이 호출할 때 사용
    핸들러(컨트롤러) 호출 방법은 타입에 따라 다르며 DispatcherServlet은 타입을 알 수 없는데
    컨트롤러 타입에 적합한 adapter를 통해 핸들러(컨트롤러)를 호출 해준다
    디폴트로 HttpRequestHandlerAdapter, SimpleControllerHandlerAdapter, AnnotationMethodHandlerAdapter 설정

    자주 사용하는 @RequestMapping, @Controller 를 통해 정의되는 컨트롤러는
    DefalutAnnotationHandlerMapping 에 의 해 핸들러 결정되고 그에 대응되는 AnnotationMethodHandlerAdapter에 의해 호출 된다

HandlerExceptionResolver

    DispatcherServlet은 HandlerExceptionResolver 중에서 발생한 예외에 적합한 것을 찾아서 예외처리 위임
    디폴트로 AnnotationMethodHandlerExceptionResolver, ResponseStatusExeptionResolver, DefaultHandlerExceptionResolver 설정

ViewResolver

    컨트롤러가 리턴한 뷰 이름을 참고해서 적절한 뷰 오브젝트를 찾는다
    디폴트로 InternalResourceViewResolver 설정, JSP나 서블릿에 리소스를 뷰로 사용
    뷰의 종류는 다양하며 뷰의 종류에 따라 적잘한 view Resolver를 추가로 설정 가능

LocaleResolver

    지역정보 결정 전략, 디폴트로 AcceptHeaderLocaleResolver 설정
    Http 헤더 정보를 보고 지역정보 설정, 세션 or URL 파라미터, 쿠키 등 다양한 방식으로 설정 변경 가능

ThemeResolver

    테마를 통해 사이트를 구성하는 케이스 잘 사용되지 않는다

RequestToViewNameTranslator

    컨트롤러에서 이름을 제공하지 않은 경우 요청정보로 뷰 이름을 자동 생성해주는 전략
    디폴트로 DefaultRequestToViewNameTranslator 설정

### @MVC

---

@MVC 의 특징으로 핸들러 매핑과 핸들러 어댑터의 대상은 오브젝트가 아닌 메소드 이다

#### @RequestMapping 핸들러 매핑

    @MVC는 디폴트로 DafaultAnnotationHandlerMapping 설정 되어 있으며 이 핸들러 매핑은 @RequestMapping 어노테이션을 활용한다
    @RequestMapping 은 타입 레벨, 메소드 레벨을 설정할 수 있고 2가지 매핑정보를 활용하여 최종 매핑정보 생성 한다
    타입 레벨의 @RequestMapping 정보를 기준하고 메소드 레벨의 매핑은 더 세분화시 사용

@RequestMapping 어노테이션

* @RequestMapping 엘리먼트
  * String[] value(): URL 패턴
  * RequestMethod[] method(): HTTP 요청 메소드
  * String[] params(): 요청 파라미터
  * String[] headers(): HTTP 헤더

String[] value(): URL 패턴

    디폴트 엘리먼트이며 스트링 배열 타입으로 URL 패턴을 지정한다
    ANT 스타일의 와일드카드를 사용할 수 있다 ex) @RequestMapping("/admin/**/user")
    {} 를 사용하는 URI 템플릿을 사용 할 수 있으며 {userid} 안에 들어가는 속성을 패스 변수라고 부른다
    @RequestMapping({/hello", "/hi"}) 하나 이상의 URL 패턴도 지정 가능
    URL 패턴은 디폴트 접미어 패턴이 적용된다
    ex) @RequestMapping("/hello") >> @RequestMapping({"/hello", "/hello/", "/hello.*"}) 와 같다

RequestMethod[] method(): HTTP 요청 메소드

    HTTP 메소드를 정의한 ENUM
    GET, HEAD, POST, PUT, DELETE, OPTIONS, TRACE 7개의 HTTP 메소드 정의
    URL이 같아도 메소드에 따라 호출되는 메소드가 다르도록 매핑할 수 있다
    ex) @RequestMapping(value="/user/add". method=RequestMethod.GET)
    ex) @RequestMapping(value="/user/add". method=RequestMethod.POST)

String[] params(): 요청 파라미터

    타입=값 형식으로 매핑할 수 있다

String[] headers(): HTTP 헤더

    HTTP 헤더 매핑이며 아래와 같이 매핑시킬수 있다
    ex) @Requestmapping(value ="/view", headers="content-type=text/*")

타입 레벨 매핑과 메소드 레벨 매핑의 결합

    타입(클래스, 인터페이스) 레벨에 붙는 @RequestMapping은 타입 내의 모든 매핑 메소드의 공통 조건 지정으로 사용
    메소드 레벨의 매핑은 세분화로 사용


```java
//두 개의 매핑 결합 예
@RequestMapping("/ctrl")
public class controller {
    
    @RequestMapping("/add")
    public String add() {
        return "";
    }

    @RequestMapping("/edit")
    public String edit() {
        return "";
    }
}
```

메소드 레벨 단독 매핑

    메소드 레벨에 독립적으로 매핑정보를 지정 하면 타입 레벨에는 조건이 없는 @RequestMapping 을 붙이면 된다
    타입 레벨에 @RequestMapping 을 적용하지 않으면 해당 클래스는 매핑 대상이 되지 않아서 메소드의 매핑도 무시된다
    그러나 컨틀롤러 클래스에 @Controller 어노테이션을 붙이면 @RequestMapping 을 생략할 수 있다

```java
//메소드 레벨의 매핑만 사용하는 예
@Controller
public class controller {
    
    @RequestMapping("/add")
    public String add() {
        return "";
    }

    @RequestMapping("/edit")
    public String edit() {
        return "";
    }
}
```

타입 레벨 단독 매핑

    @RequestMapping 을 타입에 지정할 떄 ex) @RequestMapping("/user/*") 로 입력하고
    메소드 단위에 입력 값 없는 @RequestMapping을 사용하면 method name을 사용하여 매핑 한다