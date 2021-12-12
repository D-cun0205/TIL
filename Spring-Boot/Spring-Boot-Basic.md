### 스프링 부트

---

Spring Boot 를 사용하기 위해서는 JDK 8 이상 부터 사용할 수 있다

    Spring Boot 가 지원하는 의존성 관리
    version 을 입력하지 않으면 spring-boot-dependecies.pom 에 버전을 상속받아서 사용한다
    상속구조 : pom.xml -> spring-boot-starter-parent -> spring-boot-dependencies(최상위)
    장점 : 버전 관리 문제로 인해 생기는 이슈들을 모두 해결해 준다
    단점 : 자동으로 인해 사용하지 않은 라이브러리까지 상속 받을 수 있다

```xml
<!-- 
    버전을 명시하여 바꿔주고 싶을 때 properties 안에 직접 버전을 명시 해놓으면
    pom.xml 의 설정 값의 버전으로 라이브러리를 관리해준다
    
    종속성, 플러그인 차이
    dependency : 프로젝트가 특정 시점(컴파일, 런타임)에 클래스 경로에서 사용할 수 있는 아티팩트(jar)
    plugin : 프로젝트 빌드 중 작업이 수행하도록 구성하는 아티팩트(jar)이며 클래스 경로에서 사용할 수 없다
-->
<properties>
    <spring.version>5.0.6.RELEASE</spring.version>
</properties>
```