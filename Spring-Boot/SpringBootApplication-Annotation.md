### @SpringBootApplication

---

    @SpringBootApplication 이 설정된 클래스를 메인 어플리케이션 클래스라고 부른다
    스프링이 권장하는 @SpringBootApplication 이 설정되어 있는 클래스는 최상위 패키지 바로 아래를 요구
    스프링 부트에서는 컴포넌트 스캔의 시작이 @SpringBootApplication 클래스 를 기준으로 하위를 스캔하기 때문
    @SpringBootApplication 의 주요 어노테이션
        @SpringBootConfiguration : @Configuration 과 같다, 자바 설정 파일
        @EnableAutoConfiguration : @ComponentScan 의 빈 등록 기능, 웹 어플리케이션을 구동하기 위한 설정
        @ComponentScan : IOC 컨테이너에 빈으로 등록해주기 위해 @Component 가 붙은 클래스들을 찾아서 빈 등록

    mvn -> spring-boot-autoconfigure > spring.factories 파일 안에
    @SpringBootApplication 어노테이션을 사용하면 많은 클래스가 같이 빈으로 등록 되는데
    빈 등록되는 시점에 빈 등록 대상 클래스의 @ConditionalOnWebApplcation 어노테이션 속성 값으로
    구분하여 빈으로 등록 해준다

```java
//웹어플리케이션 설정 제거하고 어플리케이션으로 실행
public class Configuration {
    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(Application.class);
        springApplication.setWebApplicationType(WebApplicationType.NONE);
        springApplication.run(args);
    }
}
```