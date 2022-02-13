### @RedirectAttribute

---

    스프링 프레임워크에서 리다이렉트(return "redirect:/뷰네임")는 Model.addAttribute 로 
    입력한 파라미터 타입 중 primitive 타입은 URI 쿼리 매개변수로 등록하는데
    스프링부트에서는 WebMvcAutoConfiguration.requestMappingHandlerAdapter 메소드를 통해
    adapter 에 isIgnoreDefaultModelOnRedirect = false 로 설정하여 URI 매개변수로 등록하지 않도록 설정한다
    그러므로 아래와 같은 요청의 응답으로 URI에 name, limit 매개변수가 보여져야 하지만
    스프링부트에서는 해당 기능이 비활성화 되어 있으므로 true 로 설정하여 활성화할 수 있다
    * RedirectAttribute 는 URI 매개변수로 등록되기 때문에 문자열만 입력 가능하므로 객체를 넣을 수 없다

```java
@Controller
@RequestMapping("/events")
@SessionAttributes("events")
public class SampleController {
    @PostMapping("/form/limit")
    public String getLimitForm(
            @ModelAttribute Events events,
            BindingResult bindingResult,
            @SessionAttribute LocalDateTime localDateTime,
            Model model)
    {
        if (bindingResult.hasErrors())
            return "/events/form-limit";
        model.addAttribute("name", events.getName());
        model.addAttribute("limit", events.getLimit());
        return "redirect:/events/list";
    }
}
```

리다이렉트 프로퍼티 활성화

```properties
spring.mvc.ignore-default-model-on-redirect=false
```

활성화 전 응답

    http://localhost:8080/events/list

활성화 후 응답

    http://localhost:8080/events/list?name=스프링&limit=50

특정 파라미터만 리다이렉트 URI 매개변수로 설정하는 방법

    properties 에 설정값은 필요 없으므로 제거
    메소드 매개변수로 RedirectAttibutes 타입의 변수를 선언하여 특정 값을 설정
    아래는 변경한 소스코드와 출력 결과

```java
@Controller
@RequestMapping("/events")
@SessionAttributes("events")
public class SampleController {
    @PostMapping("/form/limit")
    public String getLimitForm(
            @ModelAttribute Events events,
            BindingResult bindingResult,
            @SessionAttribute LocalDateTime localDateTime,
            RedirectAttributes redirectAttributes)
    {
        if (bindingResult.hasErrors())
            return "/events/form-limit";
        redirectAttributes.addAttribute("name", events.getName());
        return "redirect:/events/list";
    }
}
```

출력 결과

    http://localhost:8080/events/list?name=스프링

위와 같이 전달한 URI 파라미터를 받는 두가지 방법 중 첫번째 @RequestParam

```java
@Controller
@RequestMapping("/events")
@SessionAttributes("events")
public class SampleController {
    @GetMapping("/list")
    public String listEvent(
            @RequestParam(name = "name") String name,
            @RequestParam(name = "limit") Integer limit,
            HttpSession httpSession,
            Model model) {
        LocalDateTime localDateTime = (LocalDateTime) httpSession.getAttribute("localDateTime");
        System.out.println("time : " + localDateTime + " name : " + name + " limit " + limit);

        Events newEvents = new Events();
        newEvents.setName(name);
        newEvents.setLimit(limit);

        List<Events> list = Arrays.asList(newEvents);
        model.addAttribute("eventList", list);
        return "events/list";
    }
}
```

두번째 @ModelAttribute

    주의할 점으로 기존에 @ModelAttribute 를 사용할 때는 name 값을 지정하지 않았는데
    아래 소스코드는 name을 명시하였다 그 이유는 redirect 하려는 핸들러 메소드 안에
    SessionStatus.setComplete() 와 같은 세션 값을 제거하는 코드 그리고
    @ModelAttribute 변수와 @SessionAttributes 속성 값이 같으면 redirect 요청을 받은
    핸들러에서 세션값을 @ModelAttribute 에 설정하지만 값이 비어있어 에러가 발생한다
    에러 내용 : 500 (Expected session attribute 'events')

```java
@Controller
@RequestMapping("/events")
@SessionAttributes("events")
public class SampleController {
    @GetMapping("/list")
    public String listEvent(
            @ModelAttribute(name = "notSessionEvents") Events events,
            HttpSession httpSession,
            Model model) {
        LocalDateTime localDateTime = (LocalDateTime) httpSession.getAttribute("localDateTime");
        System.out.println("time : " + localDateTime + " name : " + events.getName() + " limit " + events.getLimit());

        Events newEvents = new Events();
        newEvents.setName(events.getName());
        newEvents.setLimit(events.getLimit());

        List<Events> list = Arrays.asList(newEvents);
        model.addAttribute("eventList", list);
        return "events/list";
    }
}
```