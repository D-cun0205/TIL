### 외부 설정

---

application.properties 를 사용한 외부 설정 방법

    main, test 안에 application.properties 설정이 다르고
    test 안에 설정되지 않은 키 값을 호출하는 곳이 있으면 오류가 발생
    이유는 빌드 시 테스트안에 application.properties 가 우선순위가 더 높다

    application.properties 를 사용하는 경우 파일 위치에 따른 우선순위를 가진다
    1 : test -> resources -> config -> application.properties
    2 : test -> resources -> application.properties
    3 : main -> resources -> config -> application.properties
    4 : main -> resources -> application.properties
    빌드 시 양쪽에 파일이 있는 경우 테스트의 외부 설정 파일이 덮어쓰니 주의

```properties
notebook.name=mac
macos.version=11.5.2
# 랜덤 숫자 리, 범위도 설정 가ㅇ
random.num=${random.int}
```

```java
public class useProperties {
    @Value(${notebook.name})
    private String name;
    @Value(${macos.version})
    private String macosVer;
}
```

Environment 사용 방법

```java
public class useProperties {
    @Autowired
    Environment environment;
    
    public void test() {
        environment.getProperty("외부 설정 파일 안에 키 값");
    }
}
```

@TestPropertySource 사용 방법

```java
@TestPropertySource(
        locations = {"classpath:test.properties"},
        properties = { "key=value" })
public class Test {
    
}
```

타입-세이프 프로퍼티 @ConfigurationProperties 사용 방법

    프로퍼티를 바인딩하기 위한 객체 생성하고 @ConfigurationProperties("키값") 설정
    @EnableConfigurationProperties 로 설정 정보를 담은 클래스를 빈으로 등록
    사용방법은 기존 빈 의존성 주입 하는 방법으로 사용

```properties
notebook.name=mac
notebook.age=2
```

```java
@ConfigurationProperties("notebook")
public class ConfigurationPropertiesClass {
    private String name;
    private int age;
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}

@SpringBootApplication
@EnableConfigurationProperties(ConfigurationPropertiesClass.class)
public class SpringInitApplication {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(SpringInitApplication.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        application.run(args);
    }
}

@Component
public class SampleRunner implements ApplicationRunner {
    @Autowired
    ConfigurationPropertiesClass propertiesClass;
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("name : " + propertiesClass.getName());
        System.out.println("age : " + propertiesClass.getAge());
    }
}
```

프로파일 사용 방법

    Configuration 를 설정한 

```properties
spring.profiles.active=test
```

```java
@Profile("prod")
@Configuration
public class ProdConfiguration {
    @Bean
    public String hello() { return "prod hello"; }
}

@Profile("test")
@Configuration
public class TestConfiguration {
    @Bean
    public String hello() { return "test hello"; }
}

@Component
public class SampleRunner implements ApplicationRunner {
    @Autowired String hello;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(hello);
        //출력 결과 : test hello
    }
}
```