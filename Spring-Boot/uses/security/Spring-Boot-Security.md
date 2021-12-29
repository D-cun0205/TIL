### 스프링 부트 시큐리티

---

    스프링 부트 시큐리티를 적용하고 UserDetailsServiceAutoConfiguration -> @ConditionalOnMissingBean
    AuthenticationManager, AuthenticationProvider, UserDetailsService 이 중 하나라도 구현한 클래스가 없으면
    InmemoryUserDetailsManager 에서 인증할 수 있는 랜덤한 유저를 인메모리로 생성하고 패스워드를 스택트레이스에 출력한다
    디폴트 아이디는 user를 사용하면 된다

시큐리티, 시큐리티 테스트 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
    <version>${spring-security.version}</version>
</dependency>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
Index
<a href="/hello">hello</a>
<a href="/my">my</a>
</body>
</html>

<!-- index, hello, my html -->
```

```java
@Controller
public class HelloController  {
    
    @GetMapping("/hello")
    public String hello() {
        return "hello";
    }

    @GetMapping("/my")
    public String my() {
        return "my";
    }
}

@RunWith(SpringRunner.class)
@WebMvcTest(HelloController.class)
public class SpringDemoApplicationTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    @WithMockUser //가짜 인증 유저 (스프링 시큐리티 테스트 라이브러리)
    public void hello() throws Exception {
        mockMvc.perform(get("/hello").accept(MediaType.TEXT_HTML))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("hello"));
    }

    @Test
    public void hello_without_user() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser
    public void my() throws Exception {
        mockMvc.perform(get("/my"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(view().name("my"));
    }
}
```

### 시큐리티 설정 커스터마이징

JPA, h2(인메모리DB) 의존성 추가

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                //"/", "/hello 핸들러 요청은 인증 없이 사용
                .antMatchers("/", "/hello").permitAll()
                //나머지 요청에 대해서는 인증 후 사용
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }

    //passwordEncoder 사용하지 않고 등록한 유저로 로그인할 경우 에러 발생 및 로그인 불가
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}


//유저 등록
@Component
public class AccountRunner implements ApplicationRunner {

    @Autowired
    AccountService accountService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        accountService.createAccount("spring", "1234");
    }
}


@Service
public class AccountService implements UserDetailsService {

    @Autowired
    AccountRepository repository;

    @Autowired
    PasswordEncoder passwordEncoder;

    public Account createAccount(String username, String password) {
        Account account = new Account();
        account.setUsername(username);
        account.setPassword(passwordEncoder.encode(password));
        return repository.save(account);
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Account> byUsername = repository.findByUsername(username);
        Account account = byUsername.orElseThrow(() -> new UsernameNotFoundException(username));
        return new User(account.getUsername(), account.getPassword(), authorities());
    }

    private Collection<? extends GrantedAuthority> authorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }

}


public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUsername(String username);
}
```