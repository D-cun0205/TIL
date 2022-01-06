### 핸들러

---

#### Handler Intercepter

    핸들러 매핑(어떠한 요청을 처리할 핸들러를 찾아주는 역할)에 설정할 수 있는 인터셉터
    핸들러 실행 전, 후(렌더링 전), 완료(렌더링 후) 시점에 부가작업 추가
    로깅, 인증, 체크, Locale 과 같은 반복작업을 줄이기 위해 사용


* preHandle
* postHandle
* afterCompletion


preHandle

    매개변수 : request, response, handler
    false 일 경우 preHandle 만 사용한다
    Handler 실행 전 호출
    handler 정보를 사용하여 서블릿 필터에 비해 세밀한 로직 구현 가능
    리턴값으로 다음 인터셉터를 진행할지(true) 무시할지(false) 정할 수 있다

postHandle

    매개변수 : request, response, modelAndView
    핸들러 실행이 끝나고 뷰 렌더링 전에 호출
    뷰, 모델 정보에 담을 공통작업
    동일한 인터셉터(postHandle) 이 있으면 역순으로 호출된다
    비동기 호출에 사용안됨

afterCompletion

    매개변수 : request, response, handler, ex
    요청 처리(뷰 렌더링)가 완료 후 호출
    preHandle 에서 리턴값으로 true 를 리턴한 경우에만 호출
    동일한 인터셉터(afterCompletion) 이 있으면 역순으로 호출된다
    비동기 호출에 사용안됨


핸들러 인터셉터 구현

```java
@Component
public class SampleHandlerIntercepter implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("samplePreHandle");
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("samplePostHandle");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        System.out.println("sampleAfterCompletion");
    }
}
```

핸들러 인터셉터 등록

    Interceptor 등록시 order 를 지정하지 않으면 registry.addInterceptor 등록 순으로 순서가 정해진다
    addPathPatterns, excludePathPatterns 를 등록하여 특정 핸들러에만 작업을 추가하거나 제거할 수 있다

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new SampleHandlerIntercepter()).order(0);
        registry.addInterceptor(new TwoSampleHandlerIntercepter())
                .excludePathPatterns("/hello").order(-1);
    }
}
```