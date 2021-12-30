### 스프링 REST 클라이언트

---

* RestTemplateAutoConfiguration
* WebClient

공통 - Spring Web 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```java
@RestController
public class HelloRestController {

    //InterruptedException : 중단 된 쓰레드에 대한 예외
    @GetMapping("/hello")
    public String hello() throws InterruptedException {
        Thread.sleep(5000);
        return "hello";
    }

    @GetMapping("/world")
    public String world() throws InterruptedException {
        Thread.sleep(3000);
        return "world";
    }
}
```
<br />

#### RestTemplateAutoConfiguration

    Blocking I/O, Synchronous API
    Spring-Web 의존성에 의해 빈으로 자동으로 등록되어 사용할 수 있다

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {

    @Autowired
    RestTemplateBuilder templateBuilder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        RestTemplate restTemplate = templateBuilder.build();

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        String helloResult = restTemplate.getForObject("http://localhost:8080/hello", String.class);
        System.out.println(helloResult);

        String worldResult = restTemplate.getForObject("http://localhost:8080/world", String.class);
        System.out.println(worldResult);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
``` 
<br />


#### WebClient

    Non-Blocking I/O, Asynchronous API
    Webflux 의존성에 의해 빈으로 자동으로 등록되어 사용할 수 있다

Spring Webflux 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {

    @Autowired
    WebClient.Builder webClient;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        WebClient build = webClient.build();
        Mono<String> helloMono = build.get().uri("http://localhost:8080/hello").retrieve().bodyToMono(String.class);
        helloMono.subscribe(s -> {
            System.out.println(s);

            if (stopWatch.isRunning())
                stopWatch.stop();

            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });

        Mono<String> worldMono = build.get().uri("http://localhost:8080/world").retrieve().bodyToMono(String.class);
        worldMono.subscribe(s -> {
            System.out.println(s);

            if (stopWatch.isRunning())
                stopWatch.stop();

            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```

커스터마이징

```java
@Configuration
public class RestCustomizerConfig {
    @Bean
    public WebClientCustomizer webClientCustomizer() {
        return webClientBuilder -> webClientBuilder.baseUrl("http://localhost:8080");
        //WebClient 의 요청에서 /hello, /world 입력하고 응답받을 수 있다
    }

    @Bean
    public RestTemplateCustomizer restTemplateCustomizer() {
        return restTemplate -> restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
    }
}
```