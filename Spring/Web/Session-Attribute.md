### @SessionAttribute

---

    SessionAttribute 를 SessionAttributes 처럼 사용할 수 있으나 주로
    @SessionAttributes : 세션 데이터를 @ModelAttribute 파라미터에 바인딩
    @SessionAttribute : 세션에 특정 값을 별도로 설정

세션에 특정 값 설정하는 인터셉터 및 인터셉터 등록 코드

    LocalDateTime 클래스를 이용하여 현재시간을 세션에 설정하고 핸들러에서 확인

```java
public class CustomInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession httpSession = request.getSession();
        httpSession.setAttribute("localDateTime", LocalDateTime.now());
        return true;
    }
}

@Configuration
public class CustomWebMvcConfigurer implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new CustomInterceptor());
    }
}
```

핸들러 컨트롤러 코드

```java
@Controller
@RequestMapping("/events")
@SessionAttributes("events")
public class SampleController {

    @PostMapping("/form/limit")
    public String getLimitForm(
            @Validated @ModelAttribute Events events,
            BindingResult bindingResult,
            @SessionAttribute LocalDateTime localDateTime)
    {
        if (bindingResult.hasErrors())
            return "/events/form-limit";
        System.out.println(localDateTime); //현재 시간 출력
        return "redirect:/events/list";
    }
}
```

@SessionAttributes 공부할 때 테스트 해봤어야 할 내용

    첫번째

    /events/form/name 요청을 처리하는 핸들러 메소드에 작성되어 있는 코드를 보면
    model.addAttribute("events", new Events()); 로 빈 Events 객체를 보낸다
    왜 보낼까? 궁금해서 위 코드를 삭제하고 실행해 보았다

    에러 발생 및 출력 내용
    Exception evaluating SpringEL expression: "#fields.hasErrors('name')" (template: "/events/form-name" - line 9, col 8)
    바인딩 된 객체가 없으니 name 속성으로 넘어오는 값을 벨리데이션 체크하기 위한 설정의 'name' 을 찾지 못해 에러가 발생한다

    두번째

    @ModelAttribute 의 기능은 속성값을 바인딩 하는데에만 그친다고 생각했다
    그래서 @ModelAttribute 로 바인딩한 파라미터를 Model 객체에 다시 model.addAttribute 로 설정해주어야 한다고 생각했는데
    @ModelAttribute 는 바인딩 하면서 View 에 넘겨지는 Model 역할도 한다

.. 너무 한심하고 부족하다 열심히 공부하자
