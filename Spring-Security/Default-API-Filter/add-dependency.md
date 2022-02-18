### 프로젝트 구성 및 의존성 추가

---

스프링 시큐리티 의존성 추가

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

스프링 시큐리티 의존성 추가 및 자동설정

    모든 요청의 접근에는 인증이 요구된다
    대표적인 인증 방식으로 폼 로그인, httpBasic 로그인 사용
    기본 로그인 페이지 제공 (로그인 페이지가 없는 경우)
    기본 계정 제공 (계정이 없는 경우)

```java
@RestController
public class SecurityController {

    @GetMapping("/")
    public String securityStart() {
        return "Hello";
    }
}
```

스프링 시큐리티 의존성 추가 후 위 핸들러 접근 시 로그인 화면 출력

    ID : user
    PW : Using generated security password: 랜덤 스트링 출력
    입력하면 인증절차를 거쳐 Hello 를 출력하는 화면을 확인할 수 있다