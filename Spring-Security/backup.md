Controller

```java
@Controller
public class ConfigController {

    @GetMapping("/config")
    public String config() {
        return "admin/config";
    }
}

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

@Controller
public class MessageController {
    @GetMapping(value = "/messages")
    public String messages() throws Exception {
        return "user/messages";
    }
}

@Controller
public class UserController {

    private UserService userService;
    private PasswordEncoder passwordEncoder;

    @Autowired
    UserController(UserService userService, PasswordEncoder passwordEncoder) {
        this.userService = userService;
        this.passwordEncoder = passwordEncoder;
    }

    @GetMapping(value = "/mypage")
    public String myPage() throws Exception {
        return "user/mypage";
    }

    @GetMapping(value = "/users")
    public String createUser() {
        return "user/login/register";
    }

    @PostMapping(value = "/users")
    public String createUser(AccountDto accountDto) {
        ModelMapper modelMapper = new ModelMapper();
        Account account = modelMapper.map(accountDto, Account.class);
        account.setPassword(passwordEncoder.encode(accountDto.getPassword()));
        userService.createUser(account);
        return "redirect:/";
    }
}

@Controller
public class HomeController {
    @GetMapping(value="/")
    public String home() throws Exception {
        System.out.println("home 진입");
        return "home";
    }
}
```

domain

```java
@Entity
@Data
public class Account {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
    private String password;
    private String email;
    private String age;
    private String role;
}

@Data
public class AccountDto {
    private String username;
    private String password;
    private String email;
    private String age;
    private String role;
}
```

repository

```java
public interface UserRepository extends JpaRepository<Account, Long> {
    Account findByUsername(String username);
}

```

security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.authenticationProvider(authenticationProvider());
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        return new CustomAuthenticationProvider();
    }

    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().requestMatchers(PathRequest.toStaticResources().atCommonLocations());
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/", "/users").permitAll()
                .antMatchers("/mypage").hasRole("USER")
                .antMatchers("/messages").hasRole("MANAGER")
                .antMatchers("/config").hasRole("ADMIN")
                .anyRequest().authenticated()
        .and()
                .formLogin()
                .loginPage("/login")
                .loginProcessingUrl("/login_proc")
                .defaultSuccessUrl("/")
                .permitAll()
        ;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}

public class CustomAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private PasswordEncoder passwordEncoder;

    //사용자가 입력한 ID, PASSWORD
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
        //authentication 객체가 UsernamePasswordAuthenticationToken 클래스와 일치하면 인증되도록 처리
        return UsernamePasswordAuthenticationToken.class.isAssignableFrom(authentication);
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

@Service("userService")
public class UserServiceImpl implements UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    @Override
    public void createUser(Account account) {
        userRepository.save(account);
    }
}

public interface UserService {

    void createUser(Account account);
}
```