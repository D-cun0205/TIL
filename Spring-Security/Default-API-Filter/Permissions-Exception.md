### 권한설정 & 표현식

---

    {noop} : 평문(사전적 의미로 암호화 예정인 순수 문자열) 사용
    AuthenticationManagerBuilder 클래스를 사용해서 메모리에 사용자를 설정하여 테스트
    hasRole 를 사용하여 권한을 설정하거나 access 를 사용하여 여러개의 권한 설정 가능

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("user").password("{noop}1111").roles("USER");
        auth.inMemoryAuthentication().withUser("sys").password("{noop}1111").roles("SYS", "USER");
        auth.inMemoryAuthentication().withUser("admin").password("{noop}1111").roles("ADMIN", "SYS", "USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                        .antMatchers("/user").hasRole("USER")
                        .antMatchers("/admin/pay").hasRole("ADMIN")
                        .antMatchers("/admin/**").access("hasRole('SYS') or hasRole('ADMIN')")
                .anyRequest().authenticated();

        http
                .formLogin();
    }
}
```

### 예외 처리 및 요청 캐시 필터

---

    RequestCacheAwareFilter : 캐싱 데이터를 담고있는 Request 객체를 필터에 지속적으로 전달하여 활용하는 역할
    RequestCache(HttpSessionRequestCache 구현체) : SavedRequest 객체를 담고 있다
    SavedRequest : 기존의 사용자, 익명사용자가 요청할 때 요청 정보들을 담는다

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .formLogin()
                //AuthenticationSuccessHandler 인터페이스
                .successHandler((request, response, authentication) -> {
                    RequestCache requestCache = new HttpSessionRequestCache();
                    SavedRequest savedRequest = requestCache.getRequest(request, response);
                    String redirectUrl = savedRequest.getRedirectUrl();
                    response.sendRedirect(redirectUrl);
                });

        http
                .exceptionHandling()
                //AuthenticationEntryPoint 인터페이스
                .authenticationEntryPoint((request, response, authException) -> {
                    response.sendRedirect("/login"); //사용자 정의 핸들러, 스프링이 지원하는 로그인 아님
                })
                //AccessDeniedHandler 인터페이스
                .accessDeniedHandler((request, response, accessDeniedException) -> {
                    response.sendRedirect("/denied");
                });
    }
}
```

Flow

