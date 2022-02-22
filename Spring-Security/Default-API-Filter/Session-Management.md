### sessionManagement method

---

    HttpSecurity.sessionManagement : 세션 관리 기능 작동

동시 세션 제어

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
                //세션이 유효하지 않을 때 이동할 페이지
                .invalidSessionUrl("/invalid")
                //세션 최대 허용 개수, -1 : 세션 무제한
                .maximumSessions(50)
                //true : 동시 로그인 차단, 세션이 멕시멈인 경우 로그인 불가
                //false : 세션이 멕시멈인 경우 기존 세션 제거, 새로운 접속에 세션 부여
                .maxSessionsPreventsLogin(true)
                //세션 만료시 이동할 페이지
                .expiredUrl("/expired");
    }
}
```

세션 고정 보호

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
                //로그인할 때 마다 세션아이디 변경 (디폴트)
                .sessionFixation().changeSessionId();
                //세션 고정, !!경고 : 해킹에 취약함
                //.sessionFixation().none();
                //새로운 세션을 생성하는데
                //.sessionFixation().newSession();

    }
}
```

세션 정책

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
                //스프링 시큐리티가 항상 세션 생성
                //.sessionCreationPolicy(SessionCreationPolicy.ALWAYS);
                //스프링 시큐리티가 필요 시 생성(기본값)
                //.sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
                //스프링 시큐리티가 생성하지 않지만 이미 존재하면 사용
                //.sessionCreationPolicy(SessionCreationPolicy.NEVER);
                //스프리 시큐리티가 생성하지ㅏ 않고 이미 존재해도 사용하지 않음
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```