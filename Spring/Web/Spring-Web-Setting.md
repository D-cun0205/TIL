### Spring Web MVC 설정

직접 빈으로 등록

```java
@Configuration
@ComponentScan
public class CustomConfig {
    
    @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping() {
        RequestMappingHandlerMapping requestMappingHandlerMapping = new RequestMappingHandlerMapping();
        //핸들러 매핑 우선 순위
        requestMappingHandlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
        return  requestMappingHandlerMapping;
    }

    @Bean
    public RequestMappingHandlerAdapter requestMappingHandlerAdapter() {
        RequestMappingHandlerAdapter requestMappingHandlerAdapter = new RequestMappingHandlerAdapter();
        //핸들러에 의해 매핑된 컨트롤러 메서드의 @RequestBody, @RequestParam, @PathVariable 와 같은 정의
        requestMappingHandlerAdapter.setArgumentResolvers();
        //요청, 응답에 대한 처리기 설정
        requestMappingHandlerAdapter.setMessageConverters();
        return requestMappingHandlerAdapter;
    }
}
```

디폴트 설정을 사용하지 않고 WebMvcConfigurer 를 Override 하여 직접 설정

```java
@Configuration
@ComponentScan
/*
 * 어노테이션 기반 스프링 MVC를 사용할 때 편리한 웹 MVC 기본 설정
 * DelegatingWebMvcConfiguration.class 를 Import 하고 있으며
 * 해당 클래스는 WebMvcConfigurationSupport 를 상속받고 있는데 많은 기능들이 설정 되어 있는데
 * RequestMappingHandlerMapping, Adapter 의 ORDER 설정을 최우선 순위로 하여 이와 같은 설정으로 약간의 성능 향상을 할 수 있다
 * 해당 클래스는 여러가지 기능들을 가지고 있지만 필요에 따라 네임 그대로의 위임패턴 형태로
 * 클래스패스에 특정 기능들이 추가되어 있으면 해당 기능들도 포함된 객체로 사용되어 커스텀하여 사용할 수 있다
 */
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/",".jsp");
    }
}
```

Spring Boot 에서 특정 기능들을 커스텀 하는 방법

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.tomcat.embed</groupId>
        <artifactId>tomcat-embed-jasper</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

```properties
spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

```jsp
<%@ page contentType="text/html; charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>Title</title>
</head>
<body>
    <h1>이벤트 목록</h1>
    <h2>${message}</h2>
    <table>
        <tr>
            <th>이름</th>
            <th>시작</th>
        </tr>
        <c:forEach items="${events}" var="event">
            <tr>
                <td>${event.name}</td>
                <td>${event.startTime}</td>
            </tr>
        </c:forEach>
    </table>
</body>
</html>
```

```java
public class Event {
    private String name;
    private LocalDateTime startTime;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public LocalDateTime getStartTime() {
        return startTime;
    }

    public void setStartTime(LocalDateTime startTime) {
        this.startTime = startTime;
    }
}

@Controller
public class EventController {

    @GetMapping("/events")
    public String event(Model model) {
        Event event1 = new Event();
        event1.setName("Spring");
        event1.setStartTime(LocalDateTime.of(2022, 1, 3, 22, 1));

        Event event2 = new Event();
        event2.setName("Boot");
        event2.setStartTime(LocalDateTime.of(2022, 1, 3, 22, 3));

        List<Event> events = List.of(event1, event2);
        model.addAttribute("events", events);
        model.addAttribute("message", "Happy New Year!");
        return "events/list";
    }
}
```