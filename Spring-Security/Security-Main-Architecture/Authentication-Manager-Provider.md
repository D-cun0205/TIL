### AuthenticationManager

---

AuthenticationManager 의 작업 요청 및 응답 대상 : Filter, Provider

컴파일

    AuthenticationManagerBuilder 는 ProviderManager 객체를 생성하는데
    상속 구조로 ProviderManager 를 생성하며 첫번째로 로그인 인증 방법에 따른 provider 가 등록되고
    두번째 ProviderManager 는 처음에 등록한 Manager 와 anonymousAuthenticationProvider 를 등록한다
    Form 인증 : DaoAuthenticationProvider
    RememberMe 인증 : RememberMeAuthenticationProvider
    Oauth 인증 : OauthAuthenticationProvider
    
런타임

    formLogin 인증 방법으로 진행시 UsernamePasswordAuthenticationFilter 가 요청을 받은 후
    등록되어 있는 ProviderManager 에게 UsernamePasswordAuthenticationToken 을 인수로 전달한다
    ProviderManager 는 디폴트로 anonymousAuthenticationProvider 가 등록되어 있고
    Parent 에는 DaoAuthenticationProvider 가 등록되어 있는데
    처음에는 디폴트로 등록되어있는 provider 의 토큰과 UsernamePasswordAuthenticationFilter 에서 전달한
    UsernamePasswordAuthenticationToken 을 비교하는데 디폴트로 등록되어 있는 Provider 의 토큰은
    AnonymousAuthenticationToken 이므로 다음으로 parent 에 있는 provider 를 찾는데
    부모 provider 는 UsernamePasswordAuthenticationToken 을 가지고 있으므로
    조건이 성립되면서 provider.authenticate(인증 정보) 를 호출하여 인증을 진행하고
    인증 후 요청을 전달한 필터에게 인증 완료한 객체를 반환한다
    
### AuthenticationProvider

AuthenticationProvider 의 작업 요청 및 응답 대상 : manager, UserDetailsService

컴파일

    Provider 등록은 위 AuthenticationManager 에서 설명과 같음

런타임

    formLogin 인증 방법으로 진행 UsernamePasswordAuthenticationFilter 가 요청을 받은 후
    Manager 설명에 작성되어있는 방법으로 Provider 등록
    DaoAuthenticationProvider 의 토큰과 사용자가 전달한 토큰이 같은 객체인지 확인
    확인 후 retrieveUser 메소드에서 UserDetailsService 에 사용자가 전달한 id(username) 과 같은게 있는지 확인 요청
    테스트에 사용하는 아이디는 DB 를 사용하지 않으므로 InMemoryUserDetailsManager 에서 확인 후 User 객체 리턴
    이 때 username 이 없는 사용자인 경우 UserNotFoundException 리턴
    사용자가 있으면 DaoAuthenticationProvider 에 User 객체를 리턴한다 
    User 객체를 전달받고 더 필요한 인증이 없는지 additionalAuthenticationCheck 메소드를 확인하고
    패스워드 체크 진행 패스워드를 암호화하여 서로 비교하고 같지 않으면 BadCredentialException 리턴한다
    패스워드 검증이 끝나면 다시 추가 검증(사용자가 등록한 검증)이 없는지 확인 후
    최종적으로 필터에 인증 객체를 전달하고 필터는 SecurityContextHolder 에 저장한다
