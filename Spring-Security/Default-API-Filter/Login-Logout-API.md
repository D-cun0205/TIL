### Login, Logout

---

#### Login

UsernamePasswordAuthenticationFilter

    form 로그인 시 인증 절차를 담당하는 필터

인증 절차 (Filter -> Manager -> Provider)

    AntPathRequestMatcher.class -> 요청 정보 url 매칭, 디폴트 값 "/login"(form action value 변경 가능)
    matcher 에 매칭 되지 않은 경우 chain.doFilter (다음 필터로 이동)
    matcher 에 매칭 시 User 정보를 담은 Authentication 객체 생성 후 AuthenticationManager 전달
    AuthenticationManager -> AuthenticationProvider 에 유저 정보를 담은 Authentication 객체를 전달하여 인증을 위임
    인증 실패시 AuthenticationException, 인증 성공시 Authentication 객체에 유저 정보, 권한 정보를 담아서 필터에게 리턴
    필터는 provider 에게 받은 Authentication 객체를 SecurityContext(인증 객체를 저장하는 저장소 역할) 에 저장한다
    securityContext 는 session 에 저장되어 session 에서 인증 관련 정보를 확인할 수 있다
    SuccessHandler 를 통해서 최종적으로 인증처리를 종료한다

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //인증
        http
                .authorizeRequests()
                .anyRequest().authenticated();
        //인가
        http.formLogin()
                //사용자가 생성한 로그인화면으로 이동하기 위한 핸들러로 전달
                //.loginPage("/loginPage")
                //로그인 - input - name(아이디) 명
                .usernameParameter("userId")
                //로그인 - input - password(비밀번호) 명
                .passwordParameter("passwd")
                //성공했을 때 이동하는 url
                .defaultSuccessUrl("/")
                //실패했을 때 이동하는 url
                .failureUrl("/login")
                //로그인 - 폼 - action value
                .loginProcessingUrl("/loginProc")
                //로그인 성공 핸들러
                .successHandler(new AuthenticationSuccessHandler() {
                    @Override
                    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        System.out.println("Success Message : " + authentication.getName());
                        response.sendRedirect("/");
                    }
                })
                //로그인 실패 핸들러
                .failureHandler(new AuthenticationFailureHandler() {
                    @Override
                    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
                        System.out.println("Failure Message : " + exception.getMessage());
                        response.sendRedirect("/login");
                    }
                })
                //스프링 시큐리티에서 제공하는 화면이 아닌 사용자가 정의한 화면은 모두 인증하다
                //그러므로 formLogin 설정에 대해 모두 허용하도록 permitAll 을 호출한다
                .permitAll();
    }
}
```

#### Logout

로그아웃 절차 (Filter -> LogoutHandler)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //인증
        http
                .authorizeRequests()
                .anyRequest().authenticated();

        //로그아웃 처리에서 디폴트로 post 요청, get 요청으로 처리하기 위해서는 별도의 설정 필요
        http
                .logout()
                .logoutUrl("/logout")
                .logoutSuccessUrl("/login")
                .addLogoutHandler(new LogoutHandler() {
                    @Override
                    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
                        HttpSession httpSession = request.getSession();
                        //세션 무효화
                        httpSession.invalidate();
                    }
                })
                .logoutSuccessHandler(new LogoutSuccessHandler() {
                    @Override
                    public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
                        response.sendRedirect("/login");
                    }
                })
                .deleteCookies("remember-me");
    }
}
```