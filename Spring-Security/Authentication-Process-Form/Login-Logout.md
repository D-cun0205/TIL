### Login

---

직접 생성한 login 페이지를 시큐리티에 적용하여 시큐리티에서 관리

    HttpSecurity.loginPage() : login 화면으로 전달하기 위한 핸들러 매핑
    HttpSecurity.loginProcessingUrl() : thymeleaf th:action 매핑
    HttpSecurity.defaultSuccessUrl() : 로그인 인증 후 이동할 핸들러 매핑
    HttpSecurity.permitAll() : 로그인 화면에 대한 인증 처리 패스

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) {
        http
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/login_proc")
                .defaultSuccessUrl("/")
                .permitAll();
    }
}
```

Thymeleaf Security UI 컨트롤

    xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"
    sec:authorize : 사용자의 권한에 따라 접근할 수 있는 메뉴를 동적으로 처리
    isAnonymous() : Anonymous User 인지 확인 후 boolean 리턴
    isAuthenticated() : 인증을 받은 유저인지 확인 후 boolean 리턴

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity5</artifactId>
</dependency>
```

```html
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5">
<div th:fragment="header">
    <nav class="navbar navbar-dark sticky-top bg-dark ">
        <div class="container">
            <a class="text-light" href="#"><h4>Core Spring Security</h4></a>
            <ul class="nav justify-content-end">
                <li class="nav-item" sec:authorize="isAnonymous()"><a class="nav-link text-light" th:href="@{/login}">로그인</a></li>
                <li class="nav-item" sec:authorize="isAnonymous()"><a class="nav-link text-light" th:href="@{/users}">회원가입</a></li>
                <li class="nav-item" sec:authorize="isAuthenticated()"><a class="nav-link text-light" th:href="@{/logout}">로그아웃</a></li>
                <li class="nav-item" ><a class="nav-link text-light" href="/">HOME</a></li>
            </ul>
        </div>
    </nav>
</div>
</html>
```

### Logout

---
    
    SecurityContextLogoutHandler 클래스 의 logout 메소드를 호출하여 로그아웃 처리를 진행
    SecurityContext = null, SecurityContextHolder.clearContext() 처리하고
    /login 핸들러에 redirect 하여 login 페이지로 이동한다 
    

```java
@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "user/login/login";
    }

    @GetMapping("/logout")
    public String logout(HttpServletRequest request, HttpServletResponse response) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        if (authentication != null)
            new SecurityContextLogoutHandler().logout(request, response, authentication);
        return "redirect:/login";
    }
}
```