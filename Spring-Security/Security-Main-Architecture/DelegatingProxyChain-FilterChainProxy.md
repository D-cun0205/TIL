### 위임 필터 및 필터 빈 초기화

---

#### DelegatingProxyChain

    클라이언트의 요청을 Servlet Filter 가 전달받는데 해당 필터는 Spring Bean 이 관리하는
    필터가 아니므로 Spring 에서 지원하는 기술들을 사용할 수 없다
    이를 해결하기 위해 Spring 은 Servlet Filter 를 Spring Bean 이 구현하였고
    Servlet Filter 에서 Spring Bean 이 구현한 Servlet Filter 로 요청을 전달하는
    방법이 필요한데 그역할을 DelegatingFilterProxy(Spring Bean 은 아니다) 가 담당한다

요약

    서블릿 필터는 스프링에서 정의된 빈을 주입해서 사용할 수 없다
    특정한 이름을 가진 스프링 빈을 찾아 그 빈에게 요청을 위임
        - springSecurityFilterChain 이름으로 생성된 빈을 ApplicationContext 에서 찾아 요청 위임
        - 위임만 담당하며 실제 보안처리를 진행하지 않는다

#### FilterChainProxy

    springSecurityFilterChain 이름으로 생성되는 필터 빈
    DelegatingProxyChain 이 찾는 빈이며 요청을 위임받아서 실제 보안을 처리
    스프링 시큐리티가 초기화할 때 생성되는 디폴트 필터들을 관리하고 제어
        - 스프링 시큐리티 기본 필터, 사용자 추가 필터를  담당
    FilterChainProxy 의 모든 필터에서 인증 및 인가 예외가 발생하지 않으면 (스프링)서블릿 자원에 접근

Flow

    SecurityFilterAutoConfiguration - DelegatingFilterProxy 를 springSecurityFilterChain 빈 이름으로 생성
    WebSecurityConfiguration - WebSecurity - FilterChainProxy 를 springSecurityFilterChain 빈 이름으로 생성
    컴파일이되어 모든 객체가 메모리가 올라간 후 런타임 중 클라이언트의 요청을 받게되면
    DelegatingFilterProxy - initDelegate 메소드에서 위임할 필터를 찾게 되는데 해당 필터가 FilterChainProxy 이다
    FilterChainProxy - doFilterInternal 메소드에서 인증 및 인가 필터들을 등록하고
    정적 내부 클래스인 VirtualFilterChain 에게 내용을 전달하여 모든 필터 처리를 진행한다
    
    