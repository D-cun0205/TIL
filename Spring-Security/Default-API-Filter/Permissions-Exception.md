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

    사용자의 요청을 인증, 인가 처리를 하기위해 FilterSecurityInterceptor 클래스가 가로채고
    가로챈 요청을 처리하기 위해 ExceptionTranslationFilter 에 요청 내용을 전달한다
    위 필터는 AuthenticationException(인증 예외), AccessDeniedException(인가 예외) 작업을 진행하는데

    첫번째 예시로 user 권한을 가진 인증 된 사용자가 request (/admin) 요청한 경우 접근 권한이 없으므로
    FilterSecurityInterceptor - ExceptionTranslationFilter - AccessDeniedException 
    AccessDeniedException 은 AccessDeniedHandler 를 호출하여 하나의 핸들을 처리하고
    일반적으로 처리하는 방법은 response.redirect 로 accessDenied 에 대한 처리 핸들러로 보낸다

    두번째 예시로 인증 처리하지 않은 익명 사용자, rememberMe 의 request(/admin) 요청한 경우
    FilterSecurityInterceptor - ExceptionTranslationFilter - AccessDeniedException 처리를 진행하는데
    isAnonymous, isRememberMe 분기처리에 걸려서 AuthenticationException 으로 전달된다
    AuthenticationException 은 두가지 처리를 진행하는데
    하나는 AuthenticationEntryPoint 에서 response.redirect("/login") 핸들러로 보내고
    다른 하나는 익명 사용자 또는 rememberMe 사용자의 요청 관련 정보를 세션에 저장해두고 로그인하여 인증이 완료되면
    세션에서 요청 url을 가져와서 해당 url 핸들러에 매핑한다
    
#### CSRF

    Cross Site Request Forgercy (사이트 간 요청 위조)
    security 의존성을 추가하면 csrf 가 자동으로 설정되며
    사용하고 싶지 않은 경우 HttpSecurity.csrf().disabled() 설정으로
    FilterProxyChain 에서 제외 시킬 수 있다