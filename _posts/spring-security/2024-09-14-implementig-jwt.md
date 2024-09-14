---
title: "[Spring Security] Spring Security 6.x 버전 & JWT 실습 구현하기"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/implementig-jwt

toc: true
toc_sticky: true

date: 2024-09-14
last_modified_at: 2024-09-14
---

이번 실습에서는 다음과 같은 기능들을 구현해본다.
- 로그인
- 사용자 정보 조회
- 토큰 재발급
- 로그아웃
  
해당 프로젝트의 전체 코드는 [깃허브](https://github.com/ijnooyah/jwt-spring-security)에서 볼 수 있다.

# 🔐 JWT 구현하기

`application-jwt.yml` 파일을 생성해 JWT 관련 설정을 추가한다.  
이 파일에는 다음과 같은 정보가 포함된다. (`.gitignore`에 등록해서 올리도록 하자.)
- JWT 비밀키 (실제 프로젝트에서는 환경 변수 등으로 관리해야한다.)
- 액세스 토큰 만료 시간동안
- 리프레시 토큰 만료 시간
```yml
jwt:
  secret: helloimyoonjistudyingjwtnicetomeetyounicetomeetyounicetomeetyou

  access:
    expiration: 3600000 # 1시간(60분) (1000L(ms -> s) * 60L(s -> m) * 60L(m -> h))

  refresh:
    expiration: 1209600000 #  (1000L(ms -> s) * 60L(s -> m) * 60L(m -> h) * 24L(h -> 하루) * 14(2주))
```

`application.yml` 파일에 `profiles.include: jwt`를 추가해준다.
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jwt
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
       format_sql: true
  profiles:
    include: jwt

logging.level:
  org.hibernate.SQL: debug
  org.springframework.security: debug
```

> 스프링 부트에서는 yml의 이름을 application-xxx.yml로 만들면 xxx라는 이름의 profile이 생성되어 이를 총해 관리할 수 있다. 즉, profile=xxx라는 식으로 호출하면 해당 yml 설정들을 가져올 수 있다.


JWT를 라이브러리를 사용해 구현해보자.

```gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-security'
	implementation 'org.springframework.boot:spring-boot-starter-web'

	// jwt
	implementation 'io.jsonwebtoken:jjwt-api:0.12.6'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.12.6'
	runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.12.6'

	compileOnly 'org.projectlombok:lombok'
	runtimeOnly 'com.h2database:h2'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'org.springframework.security:spring-security-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}
```

다음으로 `JwtpProvider`를 만들어보자.  
이 클래스는 토큰 생성, 토큰 유효성 검증, 토큰에서 사용자 정보 추출 기능 등을 담당한다.


```java
@Component
@RequiredArgsConstructor
@Getter
@Slf4j
public class JwtProvider {
    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.access.expiration}")
    private Long accessTokenExpirationPeriod;

    @Value("${jwt.refresh.expiration}")
    private Long refreshTokenExpirationPeriod;

    private SecretKey secretKey;

    private final CustomUserDetailsService customUserDetailsService;

    @PostConstruct
    public void init() {
        // 특정 문자열을 base64로 인코딩해서 secretKey로 사용
        secretKey = Keys.hmacShaKeyFor(Base64.getEncoder().encode(secret.getBytes()));
    }

    public String createAccessToken(String username) {
        return createToken(username, accessTokenExpirationPeriod);
    }

    public String createRefreshToken(String username) {
        return createToken(username, refreshTokenExpirationPeriod);
    }

    private String createToken(String username, Long expirationPeriod) {
        return Jwts.builder()
                .subject(username)
                .issuedAt(new Date())
                .expiration(new Date(System.currentTimeMillis() + expirationPeriod))
                .signWith(secretKey) // 자동으로 HS256 알고리즘 적용
                .compact();
    }

    public Boolean validateToken(String token) {
        try {
            Jwts.parser()
                    .verifyWith(secretKey)
                    .build()
                    .parseSignedClaims(token);
            return true;
        } catch (SecurityException e) {
            log.warn("Invalid JWT signature: {}", e.getMessage());
        } catch (MalformedJwtException e) {
            log.warn("Invalid JWT token: {}", e.getMessage());
        } catch (ExpiredJwtException e) {
            log.warn("JWT token is expired: {}", e.getMessage());
        } catch (UnsupportedJwtException e) {
            log.warn("JWT token is unsupported: {}", e.getMessage());
        } catch (IllegalArgumentException e) {
            log.warn("JWT claims string is empty: {}", e.getMessage());
        }

        return false;
    }

    public Authentication getAuthentication(String token) {
        UserDetails userDetails = customUserDetailsService.loadUserByUsername(getUsername(token));
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    public String getUsername(String token) {
        return Jwts.parser()
                .verifyWith(secretKey)
                .build()
                .parseSignedClaims(token)
                .getPayload()
                .getSubject();
    }
}
```

이제 `CustomUserDetailsService`를 구현해보자. 해당 서비스는 스프링 시큐리티에서 사용자 정보를 로드하는 역할을 한다.  
이 서비스는 DB에서 사용자 정보를 조회하고 스프링 시큐리티의 `UserDetails` 객체로 변환한다.  
이를 통해 **우리의 도메인 모델과 스프링 시큐리티의 인증 매커니즘을 연결한다.**

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByEmail(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + username));

        return new UserAdapter(user);
    }
```

이제 `UserAdapter`와 `CustomUserDetails`를 구현해보자.  
`UserAdapter` 클래스를 따로 만든 이유는 컨트롤러에서 사용하기 위해서다. (엔티티 정보를 직접 노출하지 않기 위해)
```java
@Getter
public class CustomUserDetails implements UserDetails {

    private final User user;

    public CustomUserDetails(User user) {
        this.user = user;
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(new SimpleGrantedAuthority(user.getRole().name()));
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getEmail();
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }

}
``` 

```java
@Getter @Setter
public class UserAdapter extends CustomUserDetails {
    private String email;
    private String password;
    private String role;
    private String name;

    public UserAdapter(User user) {
        super(user);
        this.email = user.getEmail();
        this.password = user.getPassword();
        this.role = user.getRole().name();
        this.name = user.getName();
    }
}
```

우리가 만든 `UserDetailsService`를 적용하기 위해 `SecurityConfig` 클래스를 수정해주자.

```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final CustomUserDetailsService customUserDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(AbstractHttpConfigurer::disable) // csrf 비활성화
                .formLogin(AbstractHttpConfigurer::disable) // form Login 비활성화
                .logout(AbstractHttpConfigurer::disable) // logout 비활성화
                .sessionManagement(session -> session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // 세션 미사용
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                        .anyRequest().authenticated())
                .userDetailsService(customUserDetailsService);
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder(); // 기본적으로 bcrypt 암호화 알고리즘의 BCryptPasswordEncoder 객체를 생성하고 사용하게 된다
    }
}
```

---
{: .style1}

회원가입은 JWT를 사용하는 코드가 없으므로 생략하겠다. 깃허브에서 보기를 바란다.
로그인을 구현해보자. 참고로 로그인할때마다 기존 리프레시 토큰을 폐기하고 새로운 리프레시토큰을 발급한다.

AuthController
```java
@PostMapping("/login")
    public ResponseEntity<TokenResponse> login(@RequestBody LoginRequest request) {
        TokenResponse tokenResponse = authService.login(request);
        return ResponseEntity.ok(tokenResponse);
    }
```

```java
@Getter @Setter
@NoArgsConstructor
@AllArgsConstructor
public class LoginRequest {
    private String email;
    private String password;
}
```

```java
@Getter @Setter
@NoArgsConstructor
public class TokenResponse {
    private String accessToken;
    private String refreshToken;

    @Builder
    public TokenResponse(String accessToken, String refreshToken) {
        this.accessToken = accessToken;
        this.refreshToken = refreshToken;
    }
}
```

`RefreshToken` 엔티티에 다음 로직을 추가해준다
```java
public RefreshToken update(String newRefreshToken, LocalDateTime expireDate, boolean isRevoked) {
    this.token = newRefreshToken;
    this.expireDate = expireDate;
    this.revoked = isRevoked;
    return this;
}
```

`AuthService`
```java
@Transactional
public TokenResponse login(LoginRequest request) {
    User user = userRepository.findByEmail(request.getEmail())
            .orElseThrow(() -> new UsernameNotFoundException("User not found with email: " + request.getEmail()));

    if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) {
        throw new BadCredentialsException("Invalid password");
    }

    // 토큰 생성
    String accessToken = jwtProvider.createAccessToken(user.getEmail());
    String refreshToken = jwtProvider.createRefreshToken(user.getEmail());


    RefreshToken newRefreshToken = refreshTokenRepository.findByUser(user)
            .map(oldRefreshToken -> oldRefreshToken.update(refreshToken, getExpireDate(), false))
            .orElseGet(() -> saveNewRefreshToken(user, refreshToken));

    return new TokenResponse(accessToken, refreshToken);
}

private RefreshToken saveNewRefreshToken(User user, String refreshToken) {
    return refreshTokenRepository.save(RefreshToken.builder()
            .token(refreshToken)
            .user(user)
            .expireDate(getExpireDate())
            .revoked(false)
            .build()
    );
}

private LocalDateTime getExpireDate() {
    return LocalDateTime.now().plusSeconds(jwtProvider.getRefreshTokenExpirationPeriod() / 1000L);
}
```

---
{: .style1}

# 🔑 로그인 기능 구현하기

다음으로는 회원이 자신의 정보를 조회하는 기능을 만들어보자.
이 기능을 구현하기 전에 `JwtAuthenticationFilter`를 만들자. JWT 토큰의 유효성을 검증하고 유효하면 해당 사용자의 인증 정보를
[SecurityContext](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontext)에 설정하기 위해 필요하다.

```java
@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtProvider jwtProvider;
    private final CustomUserDetailsService customUserDetailsService;
    private final TokenService tokenService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = resolveToken(request);

        if (token != null && jwtProvider.validateToken(token)) {
            String username = jwtProvider.getUsername(token);
            if (tokenService.isRevokedRefreshToken(username)) {
                response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Revoked token");
                return;
            }
            UserDetails userDetails = customUserDetailsService.loadUserByUsername(username);

            if (userDetails != null) {
                UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }

        filterChain.doFilter(request, response);
    }

    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (bearerToken != null && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class TokenService {
    private final UserRepository userRepository;
    private final RefreshTokenRepository refreshTokenRepository;
    private final JwtProvider jwtProvider;

    @Transactional(readOnly = true)
    public boolean isRevokedRefreshToken(String username) {
        return refreshTokenRepository.findByUserEmail(username)
                .map(RefreshToken::isRevoked)
                .orElseThrow(() -> new RuntimeException("Refresh token not found for user"));
    }

    private LocalDateTime getExpireDate() {
        return LocalDateTime.now().plusSeconds(jwtProvider.getRefreshTokenExpirationPeriod() / 1000L);
    }

}
```

`RefreshTokenRepository`에 다음 쿼리를 추가해준다.
```java
@Query("SELECT r FROM RefreshToken r WHERE r.user.email = :email")
Optional<RefreshToken> findByUserEmail(String email);
```


`SecurityConfig`에 다음과 같이 filter를 추가해준다
```java
private final CustomUserDetailsService customUserDetailsService;
private final JwtAuthenticationFilter jwtAuthenticationFilter;

.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
.authorizeHttpRequests(authorize -> authorize
        .requestMatchers("/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
        .requestMatchers("/api/v1/auth/login", "/api/v1/auth/signup").permitAll()
        .anyRequest().authenticated())
```

---
{: .style1}

# 🔄 토큰 재발급하기

다음은 토큰 재발급 해주는 코드를 만들어보자 
리프레시 토큰을 사용해  액세스 토큰, 리프레시 토큰을 재발급해준다.

`TokenController`를 만들어준다.
참고로 해당 api 테스트할때 header에 엑세스 토큰을 넣은 상태로 즉, 로그인한 상태로 바디에 리프레시 토큰값을 넣어야한다.

```java
@RestController
@RequestMapping("/api/v1/token")
@RequiredArgsConstructor
public class TokenController {

    private final TokenService tokenService;

    @PostMapping("/refresh")
    public ResponseEntity<TokenResponse> refreshToken(@RequestBody RefreshTokenRequest request) {
        TokenResponse tokenResponse = tokenService.refreshToken(request.getRefreshToken());
        return ResponseEntity.ok(tokenResponse);
    }
}
```

`TokenService`에 `refreshToken` 메서드를 만든다.
```java
@Transactional
public TokenResponse refreshToken(String refreshToken) {
    if(!jwtProvider.validateToken(refreshToken)) {
        throw new RuntimeException("Invalid refresh token");
    }

    String username = jwtProvider.getUsername(refreshToken);
    User user = userRepository.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found with email " + username));

    RefreshToken savedRefreshToken = refreshTokenRepository.findByUser(user)
            .orElseThrow(() -> new RuntimeException("Refresh token not found for user"));

    if (!savedRefreshToken.getToken().equals(refreshToken) || !savedRefreshToken.isValid() ) {
        throw new RuntimeException("Refresh token is expired or invalid");
    }

    String newAccessToken = jwtProvider.createAccessToken(username);
    String newRefreshToken = jwtProvider.createRefreshToken(username);

    savedRefreshToken.update(newRefreshToken, getExpireDate(), false);

    return new TokenResponse(newAccessToken, newRefreshToken);
}
```

`RefreshTokenRequest`
```java
@Getter @Setter
@NoArgsConstructor
@AllArgsConstructor
public class RefreshTokenRequest {
    private String refreshToken;
}
```

`RefreshToken` 엔티티에 다음 로직을 만든다.
```java
public boolean isValid() {
    return this.getExpireDate().isAfter(LocalDateTime.now()) && !revoked;
}

```

---
{: .style1}


# 🚪 로그아웃 구현하기
로그아웃 처리는 일반적으로 클라이언트 측에서 저장된 토큰을 삭제하고 서버 측에서는 해당 리프레시 토큰을 무효화하는 방식으로 구현할 수 있다. 토큰을 무효화해 `JwtAuthenticationFilter`에서 접근하지 못하도록 한다.

`AuthController`
```java
@PostMapping("/logout")
public ResponseEntity<?> logout(@AuthenticationPrincipal UserAdapter adapter) {
    authService.logout(adapter);
    return ResponseEntity.ok().body("Successfully logged out");
}
```

`AuthService`에 `logout` 로직을 추가하고 `RefreshToken` 엔티티에 `revoke` 로직을 추가해준다.

```java
@Transactional
public void logout(UserAdapter adapter) {
    RefreshToken savedRefreshToken = refreshTokenRepository.findByUser(adapter.getUser())
            .orElseThrow(() -> new RuntimeException("Refresh token not found for user"));

    savedRefreshToken.revoke();
}
```

```java
public void revoke() {
    this.revoked = true;
}
```

이렇게 하면 `JwtAuthenticationFilter`에서 `tokenService.isRevokedRefreshToken(username)` 부분에서 걸리게 된다.

---
{: .style1}

