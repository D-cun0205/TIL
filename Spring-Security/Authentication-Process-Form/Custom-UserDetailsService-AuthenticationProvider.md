### CustomUserDetailsService

---

UserDetailsSerivce 를 커스텀하여 DB 에서 조회한 사용자 정보로 인증
    
    AccountContext : Account 객체 정보를 User 타입 객체로 생성하여 인증정보 로 사용
    Account : 사용자가 입력한 정보
    User : Spring Security 에서 지원하는 인증에 사용되는 클래스
    GrantedAuthority : 권한 인터페이스
    SimpleGrantedAuthority : GrantedAuthority 인터페이스의 구현체, 문자열을 권한으로 설정

```java
@Service("userDetailService")
public class CustomUserDetailService implements UserDetailsService {
    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Account account = userRepository.findByUsername(username);

        if (account == null)
            throw new UsernameNotFoundException("usernameNotFountException");

        List<GrantedAuthority> roles = new ArrayList<>();
        roles.add(new SimpleGrantedAuthority(account.getRole()));
        return new AccountContext(account, roles);
    }
}

public class AccountContext extends User {
    private final Account account;

    public AccountContext(Account account, Collection<? extends GrantedAuthority> authorities) {
        super(account.getUsername(), account.getPassword(), authorities);
        this.account = account;
    }

    public Account getAccount() {
        return account;
    }
}

// JPA 사용자 정보 조회
public interface UserRepository extends JpaRepository<Account, Long> {
    Account findByUsername(String username);
}
```

### CustomAuthenticationProvider

---

인증 방식 : formLogin

    커스텀한 인증 처리(provider) 를 필터 체인에 등록하여 사용자가 로그인할 때 인증을 처리하도록 한다
    

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) {
        auth.authenticationProvider(authenticationProvider());
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        return new CustomAuthenticationProvider();
    }
}
```

    supports(메소드) : 클라이언트에서 전달한 인증 객체의 토큰을 비교하여 리턴

```java
public class CustomAuthenticationProvider implements AuthenticationProvider {
    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private PasswordEncoder passwordEncoder;
    
    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = authentication.getName();
        String password = (String) authentication.getCredentials();
        AccountContext accountContext = (AccountContext) userDetailsService.loadUserByUsername(username);

        if (!passwordEncoder.matches(password, accountContext.getAccount().getPassword()))
            throw new BadCredentialsException("BadCredentialsException");

        return new UsernamePasswordAuthenticationToken(accountContext.getAccount(), null, accountContext.getAuthorities());
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
    }
}
```

Flow

    ProviderManager 에서 등록하는 기본 provider 는 AnonymousAuthenticationProvider
    parent provider 는 SecurityConfig 에서 등록한 CustomAuthenticationProvider 가 등록되고
    인증 절차를 진행하는데 첫번째 등록된 provider 의 토큰은 Anonymous...Token false 로 무시되고
    다음으로 parent provider 에 있는 토큰이 Username...Token 이므로 Token 비교 후 실제 인증을
    시작하는데 CustomAuthenticationProvider 에 작성한 로직으로 인증을 진행하고
    UsernamePasswordAuthenticationToken 객체를 생성하여 ProviderManager 에게 리턴한다