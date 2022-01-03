### DispatcherServlet

---

DispatcherServlet 초기화 순서
    
    HandlerMapping : 핸들러를 찾아주는 인터페이스
        BeanNameUrlHandlerMapping : 빈 이름을 기반의 정보를 찾아주는 핸들러 매핑
        RequestMappingHandlerMapping : 어노테이션 기반의 정보를 찾아주는 핸들러 매핑
    HandlerAdapter : 핸들러를 실행하는 인터페이스
        default : HttpRequestHandlerAdapter, SimpleControllerAdapter, RequestMappingHandlerAdapter
    HandlerExceptionResolver
    ViewResolver
    ...
    
DispatcherServlet 동작 순서

    요청 분석 (Locale, Theme, Multipart)
    요청을 처리할 핸들러를 찾는다 (DispatcherServlet 이 HandlerMapping 에게 작업 위임)
    핸들러를 실행하기 위한 어댑터를 찾는다
        핸들러의 리턴값을 보고 어떻게 처리할지 판단
            뷰 이름에 해당하는 뷰를 찾아서 모델 데이터를 렌더링
            @ResponseEntity가 있으면 Converter를 사용해서 응답 본문 생성
    예외가 발생하면 예외 처리 핸들러에게 요청 처리 위임
    응답 리턴

ViewResolver 커스터마이징

    Spring MVC 가 디폴트로 사용하는 InternalResourceViewResolver 의 View 접근 경로를
    아래와 같이 설정하여 핸들러에서 접근할 때 경로를 제거하고 View name 으로 리소스에 접근할 수 있다

```java
@Configuration
@ComponentScan
public class WebConfig() {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}

```

DispathcerServlet 구성 요소

* MultipartResolver (요청 분석)
* LocaleResolver (요청 분석)
* ThemeResolver
* HandlerMapping
* HandlerAdapter
* HandlerExceptionResolvers
* RequestToViewNameTranslator
* ViewResolvers
* FlashMapManager

MultipartResolver

    파일 업로드 요청 처리에 필요한 인터페이스
    HttpServletRequest를 MultipartHttpServletRequest로 변환하여 요청의 File을 꺼낼 수 있는 API 제공
    DispatcherServlet 이 MultipartResolver 구현 객체를 가지고 있어야 클라이언트 요청의 파일을 받거나 사용할 수 있는데
    Spring Framework 는 MultipartResolver 구현체를 등록해야 사용할 수 있으며
    Spring Boot 는 구현체 StandardServletMultipartResolver 가 디폴트로 등록되어 있어 별도의 빈 등록없이 사용할 수 있다

LocaleResolver

    클라이언트의 위치(Locale)정보를 파악하는 인터페이스
    기본 전략으로 AcceptHeaderLocaleResolver 를 사용하며 클라이언트에서 보내온 accept-language 를 보고 판단한다
    LocaleContextResolver 의 다른 구현체로 Cookie, Session, Fiexd 를 사용하는 방법도 있다

ThemeResolver

    어플리케이션에 설정된 테마를 파악하고 변경할 수 있는 인터페이스
    클라이언트의 요청에 의해 테마를 변경하거나 테마 옵션을 변경할 수 있는 설정들이 필요
    디폴트로는 FiexdThemeResolver 이며 사용하지 않는다

HandlerMapping

    클라이언트의 요청을 처리할 수 있는 핸들러 메소드(Java Config : Mapping 어노테이션, XML Config : Bean name)를 찾아준다
    보통 RequestMappingHandlerMapping 을 사용한다
     
HandlerAdapter

    HandlerMapping 이 찾아낸 핸들러를 처리하는 인터페이스
    보통 RequestMappingHandlerAdapter 를 사용한다

HandlerExceptionResolver

    요청 처리 중에 발생한 에러 처리하는 인터페이스
    @ExceptionHandler 를 사용하여 핸들러를 Exception 처리할 수 있다

RequestToViewNameTranslator

    핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우 요청을 기반으로 뷰 이름을 판단하는 인터페이스
    디폴트로 등록되어 있으며 아래는 사용 예시다

```java
public class RequestToViewNameTranslator {
    //뷰를 리턴하지만 뷰에 대한 리턴값이 없다
    @GetMapping("/sample")
    public void sample() {
        //리턴 타입, 리턴 값이 비어있음 
    }
}
```

ViewResolver

    뷰 이름(String 타입의 리턴 값)에 해당하는 뷰를 찾아내는 인터페이스
    InternalResourceViewResolver 를 사용하며 디폴트로 JSP 를 사용한다

FlashMapManager

    FlashMap 인스턴스를 가져오고 저장하는 인터페이스
    FlashMap 은 주로 리다이렉션을 사용할 때 요청 매개변수를 사용하지 않고 데이터를 전달하고 정리할 때 사용

<br />

#### web.xml 을 Java Config 로 변경하여 DispatcherServlet 등록하기

```java
public class WebApplication implements WebApplicationInitializer {
    @Override
    public void onStartUp(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register("웹설정 클래스파일");
        context.refresh();
        
        DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
        servletRegistration.Dynamic app = servletContext.addServlet("app", dispatcherServlet);
        //app 으로 시작하는 모든 요청을 해당 DispatcherServlet이 받아서 처리
        app.addMapping("/app/*");
    }
}
```
