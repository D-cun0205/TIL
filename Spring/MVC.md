### Spring Web skill, MVC

---

웹 프레젠테이션 계층 : 사용자 또는 클라이언트 시스템과 연동하는 책임

* 스프링 웹 프레임 워크
  * 스프링 서블릿/스프링 MVC
  * 스프링 포틀릿  (작은 단위의 프레젠테이션 컴포넌트)


    스프링 서블릿/스프링 MVC 는 스프링이 직접 제공하는 서블릿 기반의 MVC 프레임워크다
    프론트 컨트롤러 역할의 핵심 엔진으로 DispatcherServlet를 사용하며 다양한 종류의 컨트롤러를 동시에 사용 가능

DispatcherServlet과 MVC 아키텍처

* 작업 흐름
  * DispatcherServlet의 HTTP 요청 접수
  * DispatcherServlet에서 컨트롤러로 HTTP 요청 위임
  * 컨트롤러의 모델 생성과 정보 등록
  * 컨트롤러의 결과 리턴 : 모델과 뷰
  * DispatcherServlet의 뷰 호출
  * 모델 참조
  * HTTP 응답 전달


흐름 1.

    Java Server Servlet Container는 HTTP 요청에 대한 설정으로 Spring DispatcherServlet이 할당 되어 있으면 요청을 전달

```xml
<!-- 아래의 서블릿 매핑은 URL이 /app 으로 시작하는 모든 요청을 DispatcherServlet에 전달, 특정 확장자만 매핑하는 방법도 가능 -->
<servlet-mapping>
  <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
  <url-pattern>/app/*</url-pattern>
</servlet-mapping>
```

    DispatcherServlet은 전처리 작업이 설정되어있으면 먼저 수행한다 보통 보안, 파라미터조작, 한글 디코딩...

흐름 2.

    DispatcherServlet은 URL, 파라미터 정보, HTTP 명령 정보로 어떤 컨트롤러에 작업을 위임할지 결정
    컨트롤러 선정은 DispatcherServlet의 핸들러 매핑 전략 이용
      * 핸들러 매핑 전략 : 사용자 요청을 기준으로 어떤 핸들러(컨트롤러)에게 작업을 위임할지 결정
    DispatcherServlet이 요청을 위임할 오브젝트를 찾아서 전달하기위해 메소드를 실행시키는 방법으로 어댑터 이용
      * 어댑터(오브젝트) 패턴 : 특정 클래스의 인터페이스를 클라이언트가 기대하는 다른 인터페이스 변환
    DispatcherServlet이 핸들러 어댑터에 HttpServletRequest 타입 오브젝트 전달
    핸들러 어댑터는 HttpServletRequest를 컨트롤러에서 받을 수 있는 파라미터로 변환해서 전달

흐름 3.

    사용자 요청 해석, 비즈니스 로직 수행을 위해 서비스 계층 오브젝트에 작업 위임, 결과 받아서 모델 생성
      * 모델 : 이름 , 오브젝트의 값

흐름 4.

    뷰도 결국엔 하나의 오브젝트이며 컨트롤러가 뷰 오브젝트를 직접 리턴 가능하나 사용하지 않음
    뷰의 논리적인 이름을 리턴하면 DispatcherServlet의 전략인 뷰 리졸버가 뷰 오브젝트를 생성
    최종적으로 ModelAndView에 모델, 뷰를 등록하여 어댑터를 통해 DispatcherServlet에 전달

흐름 5. 흐름 6.

    뷰 오브젝트에게 모델을 전달하고 클라이언트에게 전달할 최종 결과물 생성 요청 (HTML 생성)
    뷰가 생성되면 HttpServletResponse 오브젝트에 Set

흐름 7.

    HttpServletResponse를 돌려줄 때 후 처리기가 있으면 후속 작업을 진행하고 최종 결과를 서블릿 컨테이너에 전달
    서블릿 컨테이너는 HttpServletResponse에 담긴 정보를 HTTP 응답으로 만들어 클라이언트에게 전송하고 작업 종료