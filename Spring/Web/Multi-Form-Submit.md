### 멀티 폼 서브밋

---

    세션을 사용하여 여러화면에서 값을 입력 받고 저장을 진행할 수 있다
    
@SessionAttributes

    핸들러메소드에 @ModelAttribute 어노테이션이 명시되어 있는 인자가 있고
    @SessionAttributes, @ModelAttribute 설정 값이 동일하면
    session 에 있는 값을 파라미터(메소드/인자)에 자동으로 설정해준다
    * @RequestBody 를 사용해서 멀티 폼 서브밋을 진행하고 싶은 경우 Model 에 담아서 전달할 수 있다

```java
@Controller
@SessionAttributes("event")
public class SampleController {

    @GetMapping("/events/form/name")
    public String eventsFormNameView(Model model) {
        model.addAttribute("event", new Event());
        return "/events/form-name";
    }

    @PostMapping("/events/form/name")
    public String eventsFormName(@Valid @ModelAttribute Event event, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            return "/events/form-name";
        }
        return "redirect:/events/form/limit";
    }

    @GetMapping("/events/form/limit")
    public String eventsFormLimitView(@ModelAttribute Event event, Model model) {
        model.addAttribute("event", event);
        return "/events/form-limit";
    }

    @PostMapping("/events/form/limit")
    public String eventsFormLimit(@Valid @ModelAttribute Event event, BindingResult bindingResult, SessionStatus sessionStatus) {
        if (bindingResult.hasErrors()) {
            return "/events/form-limit";
        }
        sessionStatus.setComplete();
        
        //DB에 저장하는 로직이 들어가는 부분
        
        return "redirect:/events/list";
    }

    @GetMapping("/events/list")
    public String getEvent(Model model) {
        Event event = new Event();
        event.setName("최종목적지");
        event.setLimit(100);

        List<Event> events = new ArrayList<>();
        events.add(event);
        model.addAttribute("events", events);
        return "/events/list";
    }
}
```

```html
<!-- list 화면 -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<a href="@{/events/form}">Create New Event</a>
<div th:unless="${#lists.isEmpty(events)}">
    <ul th:each="event: ${events}">
        <p th:text="${event.name}">Event Name</p>
    </ul>
</div>
</body>
</html>


<!-- name 화면 -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events/form/name}" method="post" th:object="${event}">
    <p th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Incorrect date</p>
    Name : <input type="text" title="name" th:field="*{name}" />
    <input type="submit" value="Create">
</form>
</body>
</html>


<!-- limit 화면 -->
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events/form/limit}" method="post" th:object="${event}">
    <p th:if="${#fields.hasErrors('limit')}" th:errors="*{limit}">Incorrect date</p>
    Limit : <input type="text" title="limit" th:field="*{limit}" />
    <input type="submit" value="Create">
</form>
</body>
</html>
```

멀티 폼 서브밋 진행중 발생한 문제

    validation 의존성이 not working 하시더라 스프링 부트의 의존성 버전 호환에 의지하여 해결 

```xml
	<dependencies>
<!--        <dependency>-->
<!--            <groupId>javax.validation</groupId>-->
<!--            <artifactId>validation-api</artifactId>-->
<!--            <version>2.0.1.Final</version>-->
<!--        </dependency>-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
		</dependency>
    </dependencies>
```