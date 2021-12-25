### Exception Handler

---

    스프링 부트 예외 처리기 : BasicErrorController
    server.error.path(1), error.path(2) error(3) 순서대로 찾아서 값이 있으면 설정하고 뒷 순서는 무시 
    HTML 의 응답을 할 경우 errorHtml(), 그 외에 error() 에서 처리 한다
    
    localhost:8080/ 요청하면 정의된 핸들러가 없으므로 기본 리소스의 404.html 을 클라이언트에 출력
    SimpleController 의 customException 메소드를 주석처리하고 서버를 실행시킨 후
    localhost:8080/hello 를 요청하면 핸들러는 있지만 CustomExceptions 를 받아서 처리하는 로직이 없어서
    기본 리소스의 5xx.html 을 클라이언트에 출력한다

    ErrorViewResolver 를 사용하여 다양한 예외 처리 화면을 클라이언트에 출력할 수 있다
    
```java
@SpringBootApplication
public class SpringtestdemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringtestdemoApplication.class, args);
    }
}

@Controller
public class SimpleController {

    @GetMapping("/hello")
    public String hello() {
        throw new CustomExceptions();
    }

    //@ExceptionHandler : 예외가 발생한 요청을 처리하는 어노테이션
    //화면에 JSON 구조의 데이터를 화면에 출력한다
    @ExceptionHandler(CustomExceptions.class)
    public @ResponseBody CustomExceptionVo customException() {
        CustomExceptionVo vo = new CustomExceptionVo();
        vo.setMessage("error.app.key");
        vo.setReason("IDK IDK IDK");
        return vo;
    }
}

public class CustomExceptions extends RuntimeException {

}

public class CustomExceptionVo {
    private String message;
    private String reason;

    public String getMessage() { return message; }
    public void setMessage(String message) { this.message = message; }
    public String getReason() { return reason; }
    public void setReason(String reason) { this.reason = reason; }
}
```

파일 명 : 5xx.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>5xx Error</h1>
</body>
</html>
```

파일 명 : 404.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
404 Error
</body>
</html>
```