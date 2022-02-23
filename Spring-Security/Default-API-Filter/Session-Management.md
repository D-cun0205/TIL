### sessionManagement method

---

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

#### SessionManagementFilter

    세션 관리 : 인증 시 사용자의 세션정보 등록, 조회, 삭제를 통해 관리
    동시적 세션 제어 : 동일 계정 접속 허용 세션수 제한
    세션 고정 보호 : 인증할 때 쿠키값을 새로 생성, 변경하여 해커의 공격을 방지
    세션 생성 정책 : 항상 생성, 필요시 생성, 생성하지 않고 있을 경우 사용, 생성하지 않고 사용하지 않음

#### ConcurrentSessionFilter

    매 요청 현재 사용자 세션이 만료되어야 하는지 체크하고 세션 만료 처리

Flow

    ConcurrentSessionControlAuthenticationStrategy (1) 동시적 세션 처리 전략, 로그인 하려는 사용자의 세션 수 확인
    ChangeSessionIdAuthenticationStrategy (2) 세션 고정 보호 처리 전략, 디폴트 설정으로 변경할 세션 ID를 생성한다
    RegisterSessionAuthenticationStrategy (3) 세션 등록 전략, 변경한 세션 정보를 등록 
 
    UsernamePasswordAuthenticationFilter 에서 CompositeSessionAuthenticationStrategy 호출
    delegate.onAuthentication() 를 호출하여 delegate 에 들어있는 전략 순서대로(위에 작성 되어있는 순번) 실행
    첫번째 동시적 세션 처리 단계 세션 수 체크 (ConcurrentSessionControlAuthenticationStrategy)
    세션 수 초과, 인증 실패 전략 인 경우 SessionAuthenticationException 리턴
    세션 수 초과, 세션 만료 전략 인 경우 allowableSessionExceeded() 호출하고 session(기존 세션).expireNo() 로 만료 시킨 후 다음 전략 호출
    두번째 세션 고정 보호 단계 (ChangeSessionAuthenticationStategy : AbstractSessionFixationProtectionStrategy 추상클래스 타입)
    ChangeSessionAuthenticationStategy 클래스 super.onSessionChange() 호출 하여 기존 세션을 새로운 세션으로 변경
    세번째 세션 정보 등록 단계 (RegisterSessionAuthenticationStrategy : SessionAuthenticationStrategy 인터페이스 타입)
    this.sessionRegistry(SessionRegistry 타입 : SessionRegistryImpl).registerNewSession() 호출 하여 정보 등록
    세번째 단계까지 거친 후 기존 세션은 만료되고 기존 세션 사용자가 서버에 자원을 요청할 때 ConcurrentSessionFilter 에서 
    session.isExpired() 로 세션 만료 체크 한 후 기존 세션은 만료되었으므로 logout & response 처리 한다 