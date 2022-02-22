### AnonymousAuthenticationFilter

---

    authentication : null 일 때 인증 받지 않은 유저를 구분하기 위해
    익명 사용자 토큰을 발급하고 isAnonymous(), isAuthenticated() 로 구분하여 화면처리 한다
    익명 사용자 토큰마저 없이 필터를 거치게 되면 AccessDenineException 이 발생한다


* AnonymousAuthenticationFilter
  * 인증 사용자, 익명 사용자를 구분하기 위한 필터
* AnonymousAuthenticationToken
  * 익명 사용자 토큰 -> Authentication 객체에 저장, session 에는 저장하지 않는다
* AbstractSecurityInterceptor 
  * 최종 필터로 인가 처리 여부 확인
* AuthenticationTrustResolverImpl 
  * 익명 사용자 토큰 객체가 인증 가능한 클래스의 인스턴스 인지 확인
* ExceptionTranslationFilter
  * 필터 체인에서 생성한 AccessDeniedException, AuthenticationException 처리
  * 익명 사용자 토큰으로 로그인 화면으로 전달하는 역할
  

Flow
    
    AnonymousAuthenticationFilter.doFilter() 에서 authentication 객체를 확인하여 null 인 경우
    AnonymousAuthenticationToken(Authentication 의 자식 타입) 타입의 토큰을 생성하여 리턴한다
    AbstractSecurityInterceptor -> AuthenticationTrustResolverImpl -> ExceptionTranslationFilter 를 거쳐서
    최종적으로 익명 사용자 토큰으로 AccessDenineException 을 발생하지 않고 로그인 페이지로 이동 시킨다