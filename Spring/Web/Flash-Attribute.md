### FlashAttribute

---

    RedirectAttributes 를 사용하여 데이터를 전달할 때 사용
    RedirectAttributes.addAttribute 와 다르게 URI 에 데이터가 노출되지 않는다
    addAttribute 는 문자열 타입만 입력 가능 하며 FlashAttribute 는 객체를 넣을 수 있다
    HTTP 세션을 사용해서 데이터를 전달하는데 addFlashAttribute 를 사용하여 넣고 리다이렉트하면
    리다이렉트 요청을 받은 핸들러에서 요청 처리 후 데이터가 제거된다 (단발성)

화면 구성(Name 입력 -> Limit 입력 -> list)

    @PostMapping("/events/form/limit") 핸들러 메서드의 파라미터를 보면
    RedirectAttributes redirectAttributes 선언되어 있다
    보통 RedirectAttributes 사용은 URI에 전달할 파라미터를 설정할 때 사용하며
    RedirectAttributes.addFlashAttribute 에 정의한 파라미터는
    리다이렉트를 요청받는 Model 객체에 자동으로 추가해준다
    

```java
@Controller
@SessionAttributes("event")
public class SampleController {

    @GetMapping("/events/form/name")
    public String eventsFormNameView(
            Model model)
    {
        model.addAttribute("event", new Event());
        return "/events/form-name";
    }

    @PostMapping("/events/form/name")
    public String eventsFormName(
            @Valid @ModelAttribute Event event,
            BindingResult bindingResult)
    {
        if (bindingResult.hasErrors())
            return "/events/form-name";

        return "redirect:/events/form/limit";
    }

    @GetMapping("/events/form/limit")
    public String eventsFormLimitView(@ModelAttribute Event event, Model model)
    {
        model.addAttribute("event", event);
        return "/events/form-limit";
    }
    
    @PostMapping("/events/form/limit")
    public String eventsFormLimit(
            @Valid @ModelAttribute Event event,
            BindingResult bindingResult,
            SessionStatus sessionStatus,
            RedirectAttributes redirectAttributes)
    {
        if (bindingResult.hasErrors())
            return "/events/form-limit";

        sessionStatus.setComplete(); //세션에 있는 데이터 제거
        redirectAttributes.addFlashAttribute("event", event);
        return "redirect:/events/list";
    }

    @GetMapping("/events/list")
    public String getEvent(
        @ModelAttribute Event event,
        HttpSession httpSession,
        Model model)
    {
        System.out.println(httpSession.getAttribute("localDateTime"));

        Event newEvent = new Event();
        newEvent.setName("Spring Boot");
        newEvent.setLimit(100);

        List<Event> events = new ArrayList<>();
        events.add(newEvent);
        events.add(event);
        model.addAttribute("events", events);
        return "/events/list";
    }
}
```

Session, Redirect 를 사용하는 테스트 작성방법

```java
@WebMvcTest
class SampleControllerTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void getList() throws Exception {
        Event newEvent = new Event();
        newEvent.setName("Spring Framework");
        newEvent.setLimit(10000);
        
        mockMvc.perform(get("/events/list")
                .sessionAttr("localDateTime", LocalDateTime.now())
                .flashAttr("event", newEvent))
                    .andDo(print())
                    .andExpect(status().isOk())
                    .andExpect(xpath("//p").nodeCount(2));
    }
}
```