### Remember Me 인증

---

    RememberMeAuthenticationFilter 에서 Authentication = Null 인지 확인하고
    Null 인 경우 RememberMeAuthenticationFilter.AutoLogin 메소드에서 extractRememberMeCookie(request) 를 통해
    remember-me 쿠키가 있는지 String 타입 변수에 담아서 remember-me 쿠키 유무를 확인한다
    쿠키가 확인되면 필터에서는 RememberMeServices(인터페이스) 를 호출하며 구현체로는
    TokenBasedRememberMeServices(Default 14일, 메모리 토큰과 사용자 토큰 비교)
    PersistentTokenBasedRememberMeServices(영구적, 클라이언트 토큰과 DB 토큰 비교) 가 있다
    토큰 을 확인하여 없으면 chain.doFilter 로 다음 필터로 넘어가고
    토큰이 있는 경우 토큰을 디코딩 가능한지 확인
    클라이언트, 서버 토큰이 같은지 확인
    토큰에 유저 정보로 서버에 저장되어 있는 유저가 있는지 확인
    위의 절차가 정상적으로 진행되면 새로운 Authentication 을 생성하여 AuthenticationManager 에 전달하여 인증 처리한다

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private UserDetailsService userDetailsService;

    @Autowired
    SecurityConfig(UserDetailsService userDetailsService) {
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated();

        //쿠키 명 : remember, 쿠키 값 : 사용자 정보 + 쿠키 만료일 문자열을 암호화하여 쿠키를 응답으로 전달
        http.rememberMe()
                //element name : remember-me
                .rememberMeParameter("remember")
                //Default 14일
                .tokenValiditySeconds(3600)
                //true : rememberMe 기능이 활성화 되어있지 않아도 자동으로 rememberMe 기능 실행
                .alwaysRemember(true) 
                .userDetailsService(this.userDetailsService);
    }
}
```