### WebIgnore

---

정적 자원 검증 무시

    WebSecurity.ignoring().requestMatchers() 에 인수로
    PathRequest.toStaticResources().atCommonLocations() 호출하는데
    atCommonLocations() 는 StaticResourceLocation 클래스로 정적자원에 대한 path 를 가져오고
    가져온 정적 자원 패스 모두 ignoring 한다

    ignoring(), permitAll() 차이점
    FilterSecurityInterceptor 에서 정적 자원에 대해 검증을 시도하는데
    WebSecurity.ignoring() 에 정적 자원 경로를 설정 하게되면 검증 자체를 시도하지 않으며
    http.authorizeRequests().antMatchers().permitAll() 에 url 을 설정하면
    검증은 시도하되 permitAll 설정 확인 후 검증을 패스한다

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    public void configure(WebSecurity web) throws Exception {
        // atCommonLocations() = StaticResourceLocation 클래스에서 정적자원에 대한 경로를 가져와서 ignore
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }
}
```

### PasswordEncoder

---

비밀번호 암호화

    평문 비밀번호 암호화, 기본 포맷으로 bcrypt 사용
    encode() : 패스워드 암호화
    matches() : 패스워드 비교

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        String password = passwordEncoder().encode("1111");
        auth.inMemoryAuthentication().withUser("user").password(password).roles("USER");
        auth.inMemoryAuthentication().withUser("manager").password(password).roles("MANAGER", "USER");
        auth.inMemoryAuthentication().withUser("admin").password(password).roles("ADMIN", "MANAGER", "USER");
    }
}
```