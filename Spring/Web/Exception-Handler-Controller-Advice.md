### @ExceptionHandler, @ControllerAdvice

---

#### @ExceptionHandler

    핸들러에서 발생하는 예외를 핸들러가 직접 처리하지 않고 @ExceptionHandler 메소드가 처리
    Spring 3.x 버전에서 주로 사용하던 방법이며 클래스 단위로만 처리가 되기 때문에
    각 클래스마다 작성을 해줘야 하는 좋지 않은 코드작성법으로 인해
    Spring 5.x 버전에서는 @ControllerAdvice 를 사용하여 전역 또는 특정 클래스에서 
    여러 설정들을 공통으로 정의해서 적용시킬 수 있게 되었다

/event/list 핸들러메소드에서 limit 을 50으로 입력받을 때 강제로 NPE 발생

```java
@Controller
@SessionAttributes("event")
@RequestMapping("/event")
public class EventController {

    @ExceptionHandler(NullPointerException.class)
    public void customException(HttpServletRequest request, HttpServletResponse response) throws IOException {
        System.out.println("NullpointException !!");
        response.getWriter().write("It's Error Message");
    }

    @GetMapping("/list")
    public String listView(@ModelAttribute Event event, Model model) {
        List<Event> list = new ArrayList<>();
        list.add(event);
        if (event.getLimit() == 50) {
            throw new NullPointerException();
        }

        Event initEvent = new Event();
        initEvent.setName("Default");
        initEvent.setLimit(3000);
        list.add(initEvent);

        model.addAttribute("events", list);
        //model.addAttribute("initModel", model.asMap().get("initModel"));
        return "event/list";
    }
}
```

#### @ControllerAdvice

    예외처리, 벨리데이션 체크, 데이터 포멧 등 다양한 설정들을 전역으로 핸들링할 수 있다

```java
@ControllerAdvice(assignableTypes = {적용할클래스.class})
@RestControllerAdvice //@ControllerAdvice + @ResponseBody
@ControllerAdvice
public class AdviceController {
    @ExceptionHandler({EventException.class, RuntimeException.class})
    public String eventErrorHandler(RuntimeException exception, Model model) {
        model.addAttribute("message", "runtime error");
        return "error";
    }

    @InitBinder("event")
    public void initEventBinder(WebDataBinder webDataBinder) {
        webDataBinder.setDisallowedFields("id");
        webDataBinder.addValidators(new EventValidator());
    }

    @ModelAttribute
    public void initModel(Model model) {
        model.addAttribute("initModel", Arrays.asList("A", "B", "C"));
    }
}
```
