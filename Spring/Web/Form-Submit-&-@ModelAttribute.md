### Handler Method - Form Submit, @ModelAttribute

---

#### Form Submit

    타임리프 표현식
    @{} : URL 표현식
    ${} : variable 표현식
    *{} : selection 표현식

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Create New Event</title>
</head>
<body>
<form action="#" th:action="@{/events}" method="post" th:object="${event}">
    <input type="text" title="name" th:field="*{name}" />
    <input type="text" title="limit" th:field="*{limit}" />
    <input type="submit" value="전송">
</form>
</body>
</html>
```

```java
public class Event {
    private String name;
    private int limit;
    //getter, setter...
}

@Controller
public class SampleController {

    @GetMapping("/events/form")
    public String eventsForm(Model model) {
        Event event =  new Event();
        event.setName("Spring");
        event.setLimit(100);
        model.addAttribute("event", event);
        return "events/form";
    }


    @PostMapping("/events")
    public @ResponseBody Event getEvent(@RequestParam String name, @RequestParam int limit) {
        Event event = new Event();
        event.setName(name);
        event.setLimit(limit);
        return event;
    }
}
```

테스트

```java
@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void getEvent() throws Exception {
        mockMvc.perform(get("/events/form"))
                .andDo(print())
                .andExpect(view().name("events/form"))
                .andExpect(model().attributeExists("event"));
    }

    @Test
    public void postEvent() throws Exception {
        mockMvc.perform(post("/events")
                        .param("name", "Boot")
                        .param("limit", "50"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("Boot"));
    }
}
```

#### @ModelAttribute

    아래 소스코드는 @ModelAttribute 를 사용할 경우 제일 많이 사용되는 패턴이다 (@Valid, @ModelAttribute, BindingResult
    @Valid : 객체 필드에 조건을 만족하는지 체크
    @ModelAttribute : 단순 타입 데이터를 복합 타입 객체로 받아오거나 생성할 때 사용
    BindingResult : 바인딩이 되지 않은 파라미터에 대해 에러를 출력하지 않고 null 값을 바인딩하고 성공 처리한다    

```java
public class Event {

    private String name;
    @Min(0)
    private Integer limit;
    //getter, setter...
}

@Controller
public class SampleController {

    @PostMapping("/events")
    public @ResponseBody Event getEvent(@Valid @ModelAttribute Event event, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            System.out.println("=========================");
            for (ObjectError allError : bindingResult.getAllErrors())
                System.out.println(allError.toString());
            System.out.println("=========================");
        }
        return event;
    }
}
```

테스트

```java
@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void postEvent() throws Exception {
        mockMvc.perform(post("/events")
                        .param("name", "Boot")
                        .param("limit", "Integer 타입에 문자열 넣고 에러 발생"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("name").value("Boot"));
    }
}
```

#### @Validated

    스프링 프레임워크가 지원하는 Validated 를 사용하여 valid 할 필드를 그룹핑할 수 있다

```java
public class Event {

    interface ValidatedName {}
    interface ValidatedLimit {}

    @NotBlank(groups = ValidatedName.class)
    private String name;
    @Min(value = 0, groups = ValidatedLimit.class)
    private Integer limit;

    //getter, setter...
}

@Controller
public class SampleController {
    @PostMapping("/events")
    public @ResponseBody Event getEvent(@Validated(Event.ValidatedName.class) @ModelAttribute Event event, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            System.out.println("=========================");
            for (ObjectError allError : bindingResult.getAllErrors())
                System.out.println(allError.toString());
            System.out.println("=========================");
        }
        return event;
    }
}
```