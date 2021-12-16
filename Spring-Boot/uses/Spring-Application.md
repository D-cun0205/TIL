### 스프링 어플리케이션

---

스프링 웹 어플리케이션 타입 설정

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(Application.class);
        //springApplication.addListeners(new SimpleListener());
        //WebApplication 타입으로
        //SERVLET(Spring MVC), REACTIVE(Web Flux) 두개 같이 사용하면 SERVLET 이 타입에 설정 된다
            //Web Flux 란 논블로킹 런타임에서 리액티브 프로그래밍 할 수 있는 Web 프레임워크
        springApplication.setWebApplicationType(WebApplicationType.NONE);
        springApplication.run(args);
    }
}
```

ApplicationListener > ApplicationStartingEvent 사용 방법

    ApplicationListener<ApplicationStartingEvent> 를 설정
    ApplicationListener 구현체에 Component 어노테이션을 설정
    그리고 실행하면 스프링이 시작될 때 리스너가 작동하여
    빈으로 등록해도 onApplicationEvent 메소드에 접근하지 못한다
    그래서 아래와 같이 SpringApplication.addListeners() 인자 값으로
    구현체 SimpleListener를 생성하여 넣어주어 실행할 수 있다

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(Application.class);
        springApplication.addListeners(new SimpleListener());
        springApplication.run(args);
    }
}


public class SimpleListener implements ApplicationListener<ApplicationStartingEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartingEvent applicationStartingEvent) {
        System.out.println("===============");
        System.out.println("starting spring");
        System.out.println("===============");
    }
}
```

ApplicationListener 사용 방법

```java
@Component
public class SimpleListener implements ApplicationListener<ApplicationStartedEvent> {
    @Override
    public void onApplicationEvent(ApplicationStartedEvent applicationStartedEvent) {
        System.out.println("===============");
        System.out.println("started spring");
        System.out.println("===============");
    }
}
```

ApplicationArguments 사용 방법

```java
@Component
public class SimpleListener {
    //생성자에 파라미터가 한개이고 파라미터가 빈이면 자동 주입
    public SimpleListener(ApplicationArguments arguments) {
        //설정 : Application > Edit Configurations > configuration 수정
        //VM options
        System.out.println("foo : " + arguments.containsOption("foo"));
        //program arguments
        System.out.println("bar : " + arguments.containsOption("bar"));
    }
}
```

ApplicationRunner 사용 방법(추천)

```java
@Component
public class SimpleListener implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("foo : " + args.containsOption("foo"));
        System.out.println("bar : " + args.containsOption("bar"));
    }
}
```

CommandLineRunner 사용 방법(비추천)

```java
@Component
public class SimpleListener implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        Arrays.stream(args).forEach(System.out::println);
        //--bar
    }
}
```