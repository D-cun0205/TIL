### Authentication

---

SecurityContext

    SecurityContext < Authentication 객체 < User 객체 를 가지고 있다
    TreadLocal 에 저장되어 아무 곳에서나 참조가 가능하도록 설계
        Thread Safe 하기 때문에 멀티 쓰레드 환경에서도 다른 스레드가 참조 불가
        TreadLocal 이란 각 스레드에 고유하게 할당되는 영역
    인증이 완료되면 HttpSession 에 저장되어 어플리케이션의 전역 참조 가능

SecurityContextHolder

    SecurityContext 객체를 저장하는 역할로 3가지 방식이 있고
        MODE_THREADLOCAL : 스레드당 SecurityContext 객체 할당
        MODE_INHERITABLETHREADLOCAL : 메인, 자식 스레드 간 동일한 SecuriryContext 유지
        MODE_GLOBAL : 응용 프로그램에서 단 하나의 SecurityContext 저장
    
    SecurityContext 초기화 방법 : SecurityContextHolder.clearContext();
    Authentication 객체 얻는 방법 : SecurityContextHolder.getContext().getAuthentication()

전략 타입에 따른 인증 객체 얻는 방법

    아래 예시를 보면 "/", "/thread" 두가지 핸들러 매핑이 있으며
    순서는 (로그인 후) "/" 로 접근하여 Authentication 객체를 얻는 두가지 방법(HttpSession, SecurityContextHolder)을 확인하고 
    다음으로 "/thread" 에 접근하여 스레드에서 인증 객체를 가져오는 방법인데
    SecurityContextHolder 의 기본전략은 MODE_THREADLOCAL 이므로 스레드 세이프 하기 때문에
    첫번째 스레드가 "/" 로 접근하여 인증한 객체를 두번째 스레드가 "/thread" 를 요청해도 같은 인증객체를 확인할 수 없다
    이 방법을 해결하기 위해 상속 전략을 사용할 수 있다 (해결 방법 코드 아래)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated();

        http
                .formLogin();
    }
}

@RestController
public class SecurityController {

    @GetMapping("/")
    public String home(HttpSession session) {

        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        SecurityContext securityContext = (SecurityContext) session.getAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY);
        Authentication authentication1 = securityContext.getAuthentication();

        return "home";
    }

    @GetMapping("/thread")
    public String thread() {
        new Thread(
                () -> {
                    Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
                }
        ).start();
        return "thread";
    }
}
```

SecurityContextHolder 스레드 상속 전략

    MODE_INHERITABLETHREADLOCAL 타입을 사용하여 스레드를 상속관계로 설정하여
    위 소스코드에서 "/", "/thread" 를 순서대로 요청한 후 두번째 요청에서 생성한 스레드안에
    Authentication 객체를 확인해 보면 첫번째와 같은 Authentication 객체(같은 메모리 참조)가 들어 있는걸 확인할 수 있다

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated()
                .and()
                .formLogin();
        
        SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);
    }
}
```