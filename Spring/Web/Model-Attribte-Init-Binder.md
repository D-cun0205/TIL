### @ModelAttribute, @InitBinder

---

#### @ModelAttribute

Method Arguments : @ModelAttribute

	클라이언트에서 전달한 값이 @ModelAttribute 를 설정한 변수에 바인딩 된다

```java
@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {
    @PostMapping("/name")
    public String insName(
            @Valid @ModelAttribute Event event,
            BindingResult bindingResult)
    {
        if (bindingResult.hasErrors())
            return "event/name";
        eventValidator.validate(event, bindingResult);
        return "redirect:/event/limit";
    }
}
```

Method Level : @ModelAttribute

	@Controller, @ControllerAdvice 클래스에서 아래와 같이 선언하면 모델 객체를 초기화하여 사용할 수 있다

```java
@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {
    @ModelAttribute
    public void initModel(Model model) {
        model.addAttribute("initModel", Arrays.asList("A", "B", "C", "D", "E"));
    }    
}
```

Method Level : @ModelAttribute, @RequestMapping 을 같이 사용할 때

    메소드의 리턴 값은 Model 에 바인딩되어 전달되고 View Name 은 아래 절차에 의해 결정된다
    SpringBoot 에서 ViewNameTranslator 를 별도로 설정하지 않으면
    RequestToViewNameTranslator 의 구현체인 DefaultRequestToViewNameTranslator 가 설정되며
    해당 구현체의 getViewName 메소드를 이용하여 HttpServletRequest 에 url을
    DispatcherServlet 에 View Name 으로 등록하고 클라이언트에게 전달되어 정상적으로 처리된다

```java
@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {
    @GetMapping("/name")
    @ModelAttribute
    public Event nameView() {
        return new Event();
    }
}
```


#### @InitBinder

    클라이언트에서 전달 받은 데이터에 대한 바인딩, 검증 설정에 사용
    
WebDataBinder

    바인딩, 검증에 대한 설정을 등록하는 클래스
    WebDataBinder.setDisallowedFields() : 클라이언트에서 전달받고 싶지 않은 필드 설정
    WebDataBinder.addValidators() : 커스텀한 Validator 를 등록할 수 있다

```java
//CustomValidator, Validator(org.springframework.validation.Validator)
public class EventValidator implements Validator {

    @Override
    public boolean supports(Class<?> clazz) {
        // clazz 를 Event 인스턴스에 할당 가능여부를 확인하여 리턴
        return Event.class.isAssignableFrom(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        Event event = (Event) target;
        if (event.getName().equalsIgnoreCase("aaa")) {
            errors.rejectValue("name", "wrongValue", "the value is not allowed");
        }
    }
}

@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {
    @InitBinder("event") //벨리데이션 대상을 설정할 수 있다, 속성 제거시 해당 컨트롤러에 적용
    public void setInitBinder(WebDataBinder webDataBinder) {
        webDataBinder.setDisallowedFields("id");
        webDataBinder.addValidators(new EventValidator());
    }
}
```

InitBinder 를 사용하지 않고 Bean 으로 등록하여 사용하는 방법

```java
@Component
public class EventValidator {
    public void validate(Event target, Errors errors) {
        if (target.getName().equalsIgnoreCase("aaa")) {
            errors.rejectValue("name", "wrongValue", "the value is not allowed");
        }
    }
}

@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {
    private EventValidator eventValidator;
    public EventController(EventValidator eventValidator) {
        this.eventValidator = eventValidator;
    }

    @PostMapping("/name")
    public String insName(
            @Valid @ModelAttribute Event event,
            BindingResult bindingResult)
    {
        if (bindingResult.hasErrors()) {
            return "event/name";
        }
        eventValidator.validate(event, bindingResult);

        return "redirect:/event/limit";
    }
}
```
