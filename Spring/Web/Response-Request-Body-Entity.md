### ResponseBody, ResponseEntity, RequestBody, HttpEntity 

---

#### ResponseBody

    @RestController : @Controller + @ResponseBody
    @Controller 를 사용할 때 @ResponseBody 가 필요하면 직접 명시해야 한다
    데이터를 HttpMessageConverter 를 사용하여 응답 본문 메시지로 보낼 때 사용

#### RequestBody

    클라이언트에서 전달받은 json 타입의 문자열을 스프링에 등록되어 있는
    HttpMessageConverter 를 사용하여 요청 본문 -> 객체에 바인딩 한다
    spring boot 에서는 Json 관련 라이브러리를 등록하지 않으면 디폴트로
    Jackson2ObjectMapperBuilder 를 등록하므로 별도의 설정 없이 사용 가능하며
    Spring Framework 를 사용하는 경우는 등록 후 사용 가능하다

```java
public class Event {
    private String id;
    @NotBlank
    private String name;
    @Min(1)
    private Integer limit;
}

@Controller
public class EventApi {
    @PostMapping("/events/api")
    public @ResponseBody Event CreateEvent(@Valid @RequestBody Event event, BindingResult bindingResult) {
        if (bindingResult.hasErrors()) {
            bindingResult.getAllErrors().forEach(errors -> {
                //클라이언트에서 넘어온 데이터가 바인딩 실패할 때 에러 출력
                System.out.println(errors);
            });
        }
        return event;
    }
}
```

Postman 요청

    POST : http://localhost:9091/events/api
    Headers : Key : Content-Type, VALUE : application/json
    Body - raw : { "id" : null, "name" : "spring", "limit" : 100 }

Postman 응답

    Status: 200 OK Time: 242 ms Size : 203B
    {
        "id": null,
        "name": "spring",
        "limit": 100
    }

#### ResponseEntity

    응답 헤더 상태, 본문 을 컨트롤 하고 싶을 때 사용
    HttpEntity 를 상속받은 확장형 클래스로 HttpEntity 의 기능과 ResponseEntity 클래스 안에 선언되어 있는
    중첩 인터페이스, 중첩 인터페이스를 상속받은 중첩클래스를 사용하여 더 긴밀한 작업을 진행할 수 있다
    downloadFile 메소드의 리턴 값으로 스태틱 팩토리 메서드를 이용하여 결과를 리턴한다

    ResourceLoader : classpath 접근
    Resource : 바디 본문에 넣을 file 전달에 사용 
    Tika.detect("파일클래스") : headers.CONTENT_TYPE 에 동적으로 파일 타입을 설정하기 위해 사용 

```java
@Controller
public class FileController {

    @Autowired
    ResourceLoader resourceLoader;

    @GetMapping("/file/{fileName}")
    public ResponseEntity<Resource> downloadFile(@PathVariable String fileName) throws IOException {
        Resource resource = resourceLoader.getResource("classpath:" + fileName);
        File file = resource.getFile();
        Tika tika = new Tika();
        String type = tika.detect(file);
        return ResponseEntity.ok() 
                .header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + resource.getFilename() +  "\"")
                .header(HttpHeaders.CONTENT_TYPE, type)
                .header(HttpHeaders.CONTENT_LENGTH, String.valueOf(file.length()))
                .body(resource);
    }
}
```

#### HttpEntity

    @RequestBody 에 요청 헤더 정보를 사용할 수 있다
    아래 코드는 RequestEntity 를 이용해서 요청 Content-Type 을 확인한다

```java
@Controller
public class EventApi {
    @PostMapping("/events/api")
    public @ResponseBody Event createEvent(RequestEntity<Event> request) {
        MediaType contentType = request.getHeaders().getContentType(); //컨텐츠 타입 확인
        return request.getBody();
    }
}
```