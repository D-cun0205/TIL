### @SessionAttributes, SessionStatus

    Form 데이터를 사용하여 등록, 수정하면 값 유지에 대한 문제점이 있는데
    이에 해당하는 방법으로 히든 필드, DB 재조회, 계층 사이의 강한 결합이 있다
    앞서 얘기한 방법에는 다양한 단점이 있고 스프링이 지원하는 방법으로 해결할 수 있다

```java
@Controller
@SessionAttribute("user") //폼의 정보를 담을 모델 이름
public class UserController {
    
    @RequestMapping(value="/user/edit", method=RequestMethod.GET)
    public String form(@RequestParam int id, Model model) {
        model.addAttribute("user", userService.getUser(id));
        return "/user/edit";
    }
    
    @RequestMapping(value="/user/edit", method=RequestMethod.POST)
    public String submit(@ModelAttribute User user) {
        return "";
    }
}
```

@SessionAttributes 기능

    첫번째로 컨트롤러 메소드가 생성하는 모델정보 중에서 @SessionAttributes 에 지정한 이름과 동일한 것이 있으면 
    세션에 저장한다, 위 예시에서 model.addAttribute("user", userService.getUser(id))가 해당된다
    
    두 번째로 @ModelAttribute가 @SessionAttributes와 같은 이름으로 선언되어 있으면 세션 데이터를 넣어준다
    @SessionAttributes, @ModelAttribute 두 개의 속성 이름이 동일하면 먼저 세션에 같은 이름의 오브젝트가 
    존재하는지 확인하고 모델 오브젝트를 새로 만들거나 세션에 있는 오브젝트를 가져와 @ModelAttribute 파라미터로 대신 사용하고
    @ModelAttribute로 넘어온 입력 값들만 바인딩 한다, 위 예시에서 ModelAttribute User user가 해당 된다

    주의사항으로 @SessionAttributes 를 사용하게 되면 세션데이터는 언제 자신이 삭제되도 되는지 알 수 없다
    세션 데이터를 사용하면 직접 삭제하는 메소드를 호출해야 하며 이때 SessionStatus.setComplete() 를 사용한다
    SessionStatus 는 컨트롤러 메소드의 파라미터로 사용할 수 있는 스프링 내장 타입이다

```java
public class Class {
    public String method(SessionStatus sessionStatus) {
        //현재 컨트롤러에 의해 세션에 저장된 정보를 모두 제거
        sessionStatus.setComplete();
    }
}
```

등록 폼을 위한 @SessionAttribute 사용

    @SessionAttributes 는 신규 등록에도 사용자에게 등록을 위한 초기값을 세팅해서 보여주거나
    등록 폼에서 입력하고 저장할 때 에러가 발생해도 폼 값을 유지시켜주거나 유용하게 사용할 수 있으며
    빈 오브젝트 모델을 만들어서 돌려주는 것이 전형적인 스프링 MVC 폼 처리 방식이다

```java
@Controller
@SessionAttributes("user")
public class Class {
    @RequestMapping(value="/user/add", method=RequestMethod.GET)
    public String addForm(Model model) {
        User user = new User();
        user.setInit();
        
        model.addAttribute("user", user);
        return "user/edit";
    }
    
    @RequestMapping(value="/user/add", method=RequestMethod.POST)
    public String addSubmit(@ModelAttribute User user);
}
```