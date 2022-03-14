### AccessDecisionManager

---

    인증 정보, 요청 정보, 권한 정보를 이용해 자원접근 허용, 거부의 최종 결정 주체
    여러 개의 Voter(심의) 를 가질 수 있으며 각 각의 Voter 값을 리턴받고 결정
    최종 접근 거부 시 예외 발생

접근 결정의 세가지 유형

* AffirmativeBased
* ConsensusBased
* UnanimousBased

AffirmativeBased (Affirmative : 긍정적)

    여러개의 Voter(심의) 클래스 중 하나라도 승인의 경우 최종 판단으로 접근 허가

ConsensusBased (Consensus : 다수)

    승인, 거부 중 다수쪽에 의해 최종 판단
    승인, 거부 수가 동일하면 디폴트 : 접근허가, 디폴트 조건 변경 : allowIfEqualGrantedDeniedDecisions = false

UnanimousBased (Unanimous : 만장일치)

    여러개의 Voter(심의) 클래스 가 모두 승인인 경우에만 최종 판단으로 접근 허가

### AccessDecisionVoter

---

    판단을 심사
    Voter 가 권한 부여 과정에서 판단하는 3가지
        Authentication(인증)(user), FilterInvocation(요청)(antMatcher("/user)), ConfigAttributes(권한)(hasRole("USER"))
    결정 방식
        ACCESS_GRANTED : 접근허용(1), ACCESS_DENIED : 접근 거부(-1), ACCESS_ABSTAIN(0)
    

Flow

    AbstractSecurityInterceptor(FilterSecurityInterceptor 구현체) 추상클래스에서 
    AccessDecisionManager 에 인증, 요청, 권한 정보를 전달하며 작업 요청
    AccessDecisionManager 는 AccessDecisionVoter 에게 전달받은 정보를 다시 전달하며 심의 요청
    AccessDecisionVoter 는 구현체의 vote 메소드를 통해 심의 하여 AccessDecisionManager 에게 심의한 값 리턴
    AccessDecisionManager 는 AccessDecisionVoter 들에 전달받은 값으로 최종 판단 진행하여 필터에게 전달하며
    인가에 실패하면 throw new AccessDeniedException 처리