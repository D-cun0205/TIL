### 정적 리소스 지원

---

* 기본 리소스 위치
  * classpath:/static
  * classpath:/public
  * classpath:/resources
  * classpath:/META-INF/resources


    http 요청 : If-Modified-Since 값을 설정하여 요청
    http 응답 : Last-Modified 값으로 요청 변경 값을 감지
                변경값이 확인 되면 200, 동일하면 304

정적 자원 접근 경로 변경

    패키지 설정
    1.resources/locations/hello.html
    2.resources/hello.html
    application.properties 설정, 서버 start, localhost:8080/hello.html 요청
    1.resources/locations/hello.html 정적 리소스를 클라이언트에 출력

```properties
spring.resources.static-locations=classpath:/locations/
```

정적 자원 접근 패턴 설정

    패키지 설정
    1.resources/static/hello.html
    application.properties 설정, 서버 start, localhost:8080/hello.html 요청
    패턴에 의해 접근 실패되며 루트 뒤에 static/hello.html 입력하여 요청하면 200 처리

```properties
spring.mvc.static-path-pattern=/static/**
```

WebMvcConfigurer.addResourceHandlers 커스터마이징

    패키지 설정
    1.resources/static/hello.html
    2.resources/m/hello.html
    addResourceHandlers 를 커스터마이징, 서버 start, 
    localhost:8080/hello.html 요청시 1.resources/static/hello.html 출력
    localhost:8080/m/hello.html 요청시 2.resources/m/hello.html 출력
    기본 리소스 위치의 캐싱 설정을 핸들러를 커스터마이징 한 곳에도 사용하려면 새로 설정해주어야 한다

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/m/**")
                .addResourceLocations("classpath:/m/")
                .setCachePeriod(20);
    }
}
```