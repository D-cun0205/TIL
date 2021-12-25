### CORS

---

Cross Origin Resource Sharing (교차 출처 자원 공유)
    
    localhost/포트(8080, 18080) 2개 서버를 배포하고 18080포트에서 8080포트 에 자원 요청
    CORS 정책 위반으로 Policy 오류 메시지 리턴
    1.@CrossOrigin
    2.WebMvcConfigurer.addCorsMappings

8080 포트 서버 설정

```java
@SpringBootApplication
@RestController
public class SpringtestdemoApplication {

    @CrossOrigin("https://localhost:18080") // 클래스 단위에 설정 가능
    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }

    public static void main(String[] args) {
        SpringApplication.run(SpringtestdemoApplication.class, args);
    }
}

@Configuration
public class ConfigCORS implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**") //범위 : 전체 핸들러 접근 
                .allowedOrigins("http://localhost:18080");
    }
}
```

18080 포트 서버 설정

```xml
<!-- jquery 의존성 설정 -->
<dependency>
    <groupId>org.webjars.bower</groupId>
    <artifactId>jquery</artifactId>
    <version>3.3.1</version>
</dependency>
```

```java
@SpringBootApplication
public class SpringBootSampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootSampleApplication.class, args);
    }
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
<script src="/webjars/jquery/3.3.1/dist/jquery.min.js"></script>
<script>
    $(function() {
        $.ajax("http://localhost:8080/hello")
                .done(function() {
                    alert('success');
                })
                .fail(function() {
                    alert('fail');
                });
    });
</script>
</html>
```

    