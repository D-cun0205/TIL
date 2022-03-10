### SecurityContextPersistenceFilter

---

SecurityContext 객체의 생성, 저장, 조회 담당

    익명 사용자
        - 새로운 SecurityContext 객체를 생성한 후 SecurityContextHolder 에 저장
        - AnonymousAuthenticationFilter - AnonymousAuthenticationToken 객체를 SecurityContext 에 저장
    인증 시
        - 새로운 SecurityContext 객체를 생성한 후 SecurityContextHolder 에 저장
        - UsernamePasswordAuthenticationFilter 에서 인증 성공 후 SecurityContext 에 
            UsernamePasswordAuthentication 객체를 SecurityContext 에 저장
        - 인증 최종 완료 시 Session 에 SecurityContext 저장
    인증 후
        - Session 에서 SecurityContext 를 꺼내어 SecurityContextHolder 에 저장
        - SecurityContext 안에 Authentication 객체가 존재하면 인증 유지

    요청에 대한 최종 응답에 SecurityContextHolder.clearContext() 를 호출하여 인증객체를 초기화 한다

익명사용자 리소스 요청 

    모든 요청은 SecurityContextPersistenceFilter 를 거치며 해당 필터에서 HttpSecurityContextRepository 클래스를
    생성하고 세션에서 SecurityContext 를 확인하여 있으면 인증객체를 포함하여 다음 필터로 넘기고 없으면 인증객체를 생성하여
    다음 필터로 넘기게 되는데 익명사용자로 요청을 받았으니 ThreadLocal 에는 빈 SecurityContext 를 저장하고
    AnonymousAuthenticationFilter 에 작업을 전달한다

인증 전 사용자 리소스 요청

    익명사용자와 같은 절차로 진행하고 폼로그인의 경우 UsernamePasswordAuthenticationFilter 에 작업을 전달한다

인증 후 사용자 리소스 요청

    HttpSecurityContextRepository - loadContext 메소드에서 세션에 등록되어 있는 인증객체를 꺼내서 SecurityContext 에 저장
    Authentication -> SecurityContext -> LocalThread -> SecurityContextHolder 순으로 최종적으로 SecurityContextHolder 에
    인증 객체가 저장되어 새로운 인증 객체 생성을 진행하지 않고 다음 필터로 작업을 전달한다

인증 객체는 Session 에 저장하고 SecurityContextHolder.clearContext() 호출하여 초기화 진행