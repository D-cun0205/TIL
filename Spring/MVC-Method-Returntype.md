### @MVC 메소드 리턴 타입

* 자동 추가 모델 오브젝트와 자동생성 뷰 이름
  * @ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트
  * Map, Model, ModelMap 파라미터
  * @ModelAttribute 메소드
  * BindingResult
* ModelAndView
* String
* void
* 모델 오브젝트
* Map/Model/ModelMap
* View
* @ResponseBody

#### 자동 추가 모델 오브젝트와 자동생성 뷰 이름

    메소드 리턴 타입에 상관없이 조건만 맞으면 모델에 자동으로 추가 되는 타입

@ModelAttribute 모델 오브젝트 또는 커맨드 오브젝트

    메소드 파라미터가 @ModelAttribute, 커맨드 오브젝트 의 경우 자동으로 리턴 모델에 추가
    아래 예제는 모두 user 파라미터 오브젝트가 모델에 추가 된다
    
```java
public class Test {
  public void add(@ModelAttribute("user") User user);
  public void add(@ModelAttribute User user);
  public void add(User user);
}
```

Map, Model, ModelMap 파라미터

    3가지 타입의 파라미터를 사용하면 미리 생성된 모델 맵 오브젝트를 전달받아 오브젝트를 추가할 수 있다
    위 3가지에 추가한 오브젝트는 DispathcerServlet을 거쳐 뷰에 전달되는 모델에 자동 추가

@ModelAttribute 메소드

    아래 소스코드는 서비스 계층 오브젝트를 이용해 코드정보의 리스트를 받아서 리턴한다
    특정 정보들의 경우 정보들을 모델에 넣어서 뷰로 보내줘야 하는데
    이 어노테이션을 사용하면 자동으로 모델에 추가된다

```java
public class Class {
    @ModelAttribte("codes")
    public List<Code> codes() {
        return codeService.getAllCodes();
    }
}
```

BindingResult

    @ModelAttribute 파라미터와 함께 사용하는 BindingResult 타입의 오브젝트도 모델에 자동으로 추가
    특정 뷰에서 사용되는 커스턴 태그나 메크로에서 사용하고 잘못된 입력 값, 바인딩 오류 메시지를 생성 시 사용한다

#### 자동 추가 되지 않은 메소드 리턴 타입

ModelAndView

    스프링 2.5 버전이나 하위버전의 컨트롤러 사용 시 리턴 값에 명시해야 했다
    3.0 이상 버전의 @Controller 어노테이션을 사용하면 더 편리한 방법이 많아서 사용하지 않는다
    아래는 ModelAndView 를 사용하는 시점부터 @Controller를 사용하는 스타일로 변경하는 단계다

```java
//2.5 or 2.5 전 버전에서 사용하던 방식
public class HelloController implements Controller {
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse)
      throws Exception {
        String name = request.getParameter("hello");
        return new ModelAndView("hello.jsp").addObject("name", name);
    }
}

//위 스타일에 @MVC 스타일에 @Controller 변경
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse)
            throws Exception {
      String name = request.getParameter("hello");
      return new ModelAndView("hello").addObject("name", name);
    }
}

//위 스타일에 @Controller 스타일로 수정
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView hello(@RequestParam String name) {
        return new ModelAndView("hello").addObject("name", name);
    }
}

//위 스타일에서 모델에 값을 넣으면 자동으로 뷰에 전달되기 떄문에 아래와 같이 사용 가능하다
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public ModelAndView hello(@RequestParam String name, Model model) {
        model.addAttribute("name", name);
        return new ModelAndView("hello");
    }
}
```

String

    메소드의 리턴 타입이 스트링인 경우 뷰 이름으로 사용
    아래 예시처럼 모델은 파라미터 맵을 사용 리턴 값은 뷰 이름을 스트링 타입으로 
    사용하는게 @Controller 메소드 작성 방법이다
    
```java
//ModelAndView 마지막 예시에서 모델 객체는 자동으로 뷰에 전달되니
//리턴타입을 스트링으로 변환하고 모델정보는 모델 맵 파라미터로 가져와서 추가한다
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public String hello(@RequestParam String name, Model model) {
        model.addAttribute("name", name);
        return "hello";
    }
}
```

void

    메소드의 리턴 타입을 void로 설정하는 경우 RequestToViewNameResolver
    전략을 통한 자동 생성 뷰 이름이 사용된다
    URL과 뷰 이름을 일관되게 사용할 수 있는 경우 고려해볼수 있다

```java
//String 예시에서 void를 사용하면 더 단순하게 변경 된다
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public void hello(@RequestParam String name, Model model) {
        model.addAttribute("name", name);
    }
}
```

모델 오브젝트

    뷰 이름은 RequestToViewNameResolver 사용, 모델에 추가할 오브젝트가 하나인 경우
    아래와 같이 Model 파라미터를 사용하지 않고 모델 오브젝트를 바로 리턴하는 방법도 있다

```java
@Controller
public class HelloController {
    @RequestMapping("/hello")
    public User hello(@RequestParam int id) {
        return userService.getUser(id);
    }
}
```

Map/Model/ModelMap

    메소드 코드 안에서 직접 생성하고 리턴하는 경우 해당 오브젝트는 모델로 사용된다
    
```java
public class Class {
    //아래와 같이 Map 을 리턴하면 Map 자체를 모델 맵으로 인식하여
    //안에 엔트리 하나하나 개별적인 모델로 다시 등록한다
    public Map view(@RequestParam int id) {
        Map userMap = userService.getUserMap(id);
        return userMap;
    }
    
    //맵 타입의 모델 오브젝트를 사용하는 방법으로 위코드를 아래와 같이 변경하여 사용해야 한다
    public void view(@RequestParam int id, Model model) {
        model.addAttribute("userMap", userService.getUserMap(id));
    }
}
```

view

    뷰 이름으로 스트링 리턴타입을 사용하지 않고 뷰 오브젝트를 사용하는 경우
    아래와 같이 사용할 수 있다

```java
public class Controller {
    @Autowired
    MarshallingView xmlView;
    
    @RequestMapping("/xml")
    public View xml(@RequestParam int id) {
        return this.xmlView;
    }
}
```

@ResponseBody

    메소드가 리턴하는 오브젝트는 뷰를 통해 결과를 만들어내는 모델로 사용되는데
    위 어노테이션을 메소드 레벨에 사용하면 HTTP 응답의 메시지 본문으로 전환된