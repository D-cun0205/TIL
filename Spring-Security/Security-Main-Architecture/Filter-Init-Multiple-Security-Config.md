### 필터 초기화와 다중 설정 클래스

---

    SecurityConfig : antMatcher 로 /admin/** 요청에 대해서는 인증, 인가 처리
    SecondSecurityConfig : 모든 요청에 대해 인증, 인가 처리 하지 않음
    시큐리티 다중 설정 클래스를 등록할 때는 반드시 @Order() 로 순서를 정해주어야한다  

```java
@RestController
public class SecurityController {
    @GetMapping("/")
    public String index() {
        return "home";
    }
    @GetMapping("/admin/**")
    public String admin() {
        return "admin";
    }
}

@Configuration
@EnableWebSecurity
@Order(0)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .antMatcher("/admin/**")
                .authorizeRequests()
                .anyRequest().authenticated()
        .and()
                .httpBasic();
    }
}

@Configuration
@EnableWebSecurity
@Order(1)
class SecondSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().permitAll()
        .and()
                .formLogin();
    }
}
```

#### 필터 초기화

    시큐리티 설정 중 필터를 생성하는 부분 WebSecurity - performBuild 메소드에서 FilterChainProxy 객체를 생성하는데
    생성시 인수로 securityFilterChains 를 넣어주며 해당 인수는 시큐리티 필터, 사용자가 설정한 시큐리티 필터를 가지고 있다
    위 소스코드로 예를 들면 securityFilterChains 배열 index 0 은 /admin/** matcher 에 관한 12가지 시큐리티 설정이 있고
    배열 index 1 은 모든 요청에 인증, 인가를 처리하지 않은 필터 14가지가 등록되어 있다
    httpBasic() 인증, formLogin() 인증에 따라 필터 수가 달라진다
    httpBasic() : BasicAuthentication

주의사항

    위 소스코드의 @Order 값을 서로 변경하고 클라이언트가 /admin/** 에 대한 요청을 하면
    인증, 인가처리가 무시된채 리소스에 접근이 가능하게 되는데 그 이유는
    SecondSecurityConfig 에서 모든 요청에 대해 permitAll() 을 설정하였기 때문에
    SecurityConfig 의 .anyRequest().authenticated() 설정이 모두 무시되기 때문이다