### 사용자 정의 보안 기능 구현

---

    application.properties 에 로그인 정보를 설정
    설정하지 않으면 기본적으로 ID : user, PW : 랜덤(console 출력)

```properties
spring.security.user.name=user
spring.security.user.password=1111
```

시큐리티 설정

    @Configuration : 구성 클래스로 런타임에 요청을 처리할 수 있도록 지정하는 어노테이션
    @EnableWebSecurity : Security 를 활성화할 수 있는 여러 클래스를 Import 하고있는 어노테이션
    WebSecurityConfigurerAdapter.class : 스프링 시큐리티의 웹 보안 기능 초기화 및 설정을 담당
    HttpSecurity.class : 시큐리티의 세부적인 기능을 설정하도록 API 제공

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //인가
        http
                //승인 요청에 대한 설정
                .authorizeRequests()
                //모든 요청에 
                .anyRequest()
                //인증 요구
                .authenticated();
        //인증
        http
                //폼 로그인 사용
                .formLogin();
    }
}
```

핸들러 컨트롤러

```java
@RestController
public class SecurityController {

    @GetMapping("/")
    public String securityStart() {
        return "Hello";
    }
}
```