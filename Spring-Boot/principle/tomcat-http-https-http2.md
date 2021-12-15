### 톰캣, HTTP, HTTPS, HTTP2

    중요 : 톰캣 8.x 버전, JDK 8 을 사용하여 HTTPS 를 설정하면 별도의 추가 설정이 필요
    스프링 공식문서 75.8.3 HTTP/2 with Tomcat 참조

내장 서블릿 컨테이너 직접 생성하는 방법

```java
public class tomcat {
    public void createTomcat() {
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080);
        //Path(요청 최상위 경로), docBase(이 어플리케이션의 WAR 파일 절대경로)
        Context context = tomcat.addContext("/", "/");
        HttpServlet servlet = new HttpServlet() {
            @Override
            protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
                PrintWriter writer = resp.getWriter();
                writer.println("<html><head><title>");
                writer.println("Hey Tomcat");
                writer.println("</title></head>");
                writer.println("<body><h1>Hello Tomcat</h1></body>");
                writer.println("</html");
            }
        };

        String servletName = "helloServlet";
        tomcat.addServlet("/", servletName, servlet);
        context.addServletMappingDecoded("/hello", servletName);

        tomcat.start();
        tomcat.getServer().await();   
    }
}
```

Spring Boot 내장 톰캣 포트 확인 방법

```java
import org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext;
import org.springframework.boot.web.servlet.context.ServletWebServerInitializedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.stereotype.Component;

@Component
public class PortListener implements ApplicationListener<ServletWebServerInitializedEvent> {

    @Override
    public void onApplicationEvent(ServletWebServerInitializedEvent servletWebServerInitializedEvent) {
        ServletWebServerApplicationContext applicationContext = servletWebServerInitializedEvent.getApplicationContext();
        //ApplicationCotext 가 ServletWebServer 여서 getWebServer 를 호출할 수 있다
        System.out.println(applicationContext.getWebServer().getPort());
    }
}
```

Spring Boot 포트 설정, 랜덤 포트 설정

```properties
#포트 명시하기
server.port=9090
```

```properties
#랜덤 포트 사용하기
server.port=0
```

HTTP 커넥터 코딩 설정(커넥터를 2개 사용하여 HTTP,HTTPS 요청 다 받기)

```java
@SpringBootApplication
@RestController
public class Application {

    @GetMapping("/hello")
    public String getHello() {
        return "hello world";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
    @Bean
    public ServletWebServerFactory serverFactory() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector());
        return tomcat;
    }
    
    private Connector createStandardConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setPort("8443");
        return connector;
    }
}

```

HTTPS SSL 설정하기

    로컬에서 생성한 SSL 인증서는 이제 우회해서 들어갈 방법이 막혀버렸다..
    HTTPS 로 접속하려면 공인 SSL 인증기관에서 발급 받아서 적용해야 하지 않을까 싶다
    공인 인증기관에서 발급 받았다 치고

```properties
#HTTPS 설정
server.ssl.key-store=keystore.p12
server.ssl.key-store-password=123456
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=spring
#HTTP2 설정
server.http2.enabled=true
```