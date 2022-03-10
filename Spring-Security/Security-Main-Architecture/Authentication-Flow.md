### Authentication Flow

Client

    Request Login

UsernamePasswordAuthenticationFilter

    (인증 전)Authentication 객체를 생성하여 id, password 를 저장하여 전달

AuthenticationManager

    UsernamePasswordAuthenticationFilter 가 전달한 Authentication 객체를 관리만 담당
    인증 전 Authentication 객체를 받거나, AuthenticationProvider 가 인증 처리 한 객체를
    전달 받아서 UsernamePasswordAuthenticationFilter 에게 다시 전달해준다

AuthenticationProvider

    AuthenticationManager 에서 전달받은 Authentication 객체로 실제 인증을 담당한다
    클라이언트가 요청한 id,password 를 가지고 있는 Authentication 객체와
    UserDetailsService 에 username 에 해당하는 객체를 전달 받고 비교하여
    인증 처리 후 AuthenticationManager 에 인증 된 Authentication 객체를 전달해준다

UserDetailsService

    AuthenticationProvider 에서 username 을 전달받아서 Repository 를 통해 데이터 조회
    데이터가 있으면 UserDetails 타입으로 반환하여 전달한다

Repository

    User 데이터 조회
    Repository 에서 찾으려는 사용자가 없으면 Exception 처리를 위해
    UsernamePasswordAuthenticationFilter 가 받고 AuthenticationFailureHandler 가 처리한다
    AuthenticationFailureHandler 처리는 SecurityConfig(커스텀한 시큐리티 설정) 클래스의
    configure(HttpSecurity http) 메소드에서 직접 처리할 수 있다