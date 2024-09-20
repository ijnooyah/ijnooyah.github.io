---
title: "[Spring Security] OAuth2 실습 - Google 로그인 구현"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security, OAuth2, 실습, OAuth2 시리즈]

permalink: /spring-security/implementing-google-login

toc: true
toc_sticky: true

date: 2024-09-19
last_modified_at: 2024-09-19
---

{% include oauth2-list.md %}

---

# 🌻 구글 서비스 등록하기

[https://console.cloud.google.com/](https://console.cloud.google.com/)로 이동해서

새 프로젝트를 선택한다.

![1단계](/assets/images/posts_img/spring-security/implementing-google-login/google1.png)

이름은 원하는 대로 지으면 된다.

API 및 서비스 > 사용자 인증 정보로 이동한다.

![2단계](/assets/images/posts_img/spring-security/implementing-google-login/google2.png)


사용자 인증 정보 만들기 > OAuth 클라이언트 ID를 선택한다.

![3단계](/assets/images/posts_img/spring-security/implementing-google-login/google3.png)

클라이언트 ID가 생성되기 전에 동의 화면 구성이 필요하므로 안내에 따라 동의 화면 구성을 클릭한다.

![4단계](/assets/images/posts_img/spring-security/implementing-google-login/google4.png)

User Type은 외부를 선택해준다.

![5단계](/assets/images/posts_img/spring-security/implementing-google-login/google5.png)

필수로 요구하는 입력칸만 채워주자.

![6단계](/assets/images/posts_img/spring-security/implementing-google-login/google6.png)

범위는 email, profile, openid 선택해주자.

![7단계](/assets/images/posts_img/spring-security/implementing-google-login/google7.png)

동의 화면 구성이 끝났으면 다시 사용자 인증 정보 > Oauth 클라이언트 ID로 들어간다.

애플리케이션 유형은 웹 애플리케이션을 선택한다. 이름은 원하는 대로 지어주자. 승인된 리디렉션 URI에 http://localhost:8080/login/oauth2/code/google를 추가해주자.

![8단계](/assets/images/posts_img/spring-security/implementing-google-login/google8.png)

생성 버튼을 클릭하면 클라이언트 ID와 클라이언트 보안 비밀번호가 나온다. 이것을 저장해두자. (이정보는 다시 볼 수 있다.)

![9단계](/assets/images/posts_img/spring-security/implementing-google-login/google9.png)

이제 클라이언트 ID와 보안 비밀번호를 프로젝트에 설정해보자.

`application.yml`이 있는 디렉토리에 `application-oauth.yml` 파일을 생성해주자.

**application-oauth.yml**
```yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: my-client-id
            client-secret: my-client-secret
```

그리고 `application.yml`에 코드를 추가해준다.

**application.yml**
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/oauth2
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
     include: oauth

logging.level:
  org.hibernate.SQL: debug
  org.springframework.security: debug
```

> 스프링 부트에서는 yml의 이름을 application-xxx.yml로 만들면 xxx라는 이름의 profile이 생성되어 이를 총해 관리할 수 있다. 즉, profile=xxx라는 식으로 호출하면 해당 yml 설정들을 가져올 수 있다.

클라이언트 ID와 보안 비밀번호가 노출되지 않기 위해 `.gitignore`에 `application-oauth.yml`를 추가해 깃허브에 올라가는걸 방지하겠다.

**.gitignore**
```
application-oauth.yml
```

---

# 🍈 구글 로그인 연동하기
OAuth2 로그인을 구현하기 위한 의존성을 추가해주자. `build.gradle`에 아래 코드를 추가해준다.

```gradle
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
```

`SeucrityConfig`에 코드를 추가해주자.

`SecurityConfig`
```java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final CustomOidcUserService customOidcUserService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
        authenticationManagerBuilder.authenticationProvider(restAuthenticationProvider);
        AuthenticationManager authenticationManager = authenticationManagerBuilder.build();            // build() 는 최초 한번 만 호출해야 한다

        http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/css/**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                        .requestMatchers("/api/v1/auth/login", "/api/v1/auth/signup").permitAll()
                        .anyRequest().authenticated())
                .addFilterBefore(restAuthenticationFilter(http, authenticationManager), UsernamePasswordAuthenticationFilter.class)
                .authenticationManager(authenticationManager)
                .exceptionHandling(exception -> exception
                        .authenticationEntryPoint(new RestAuthenticationEntryPoint())
                        .accessDeniedHandler(new RestAccessDeniedHandler())
                )
                .oauth2Login(oauth2 -> oauth2
                        .userInfoEndpoint(userInfo -> userInfo
                                .oidcUserService(customOidcUserService))
                        .successHandler(((request, response, authentication) -> {
                            response.sendRedirect("/");
                        })) // 추가 코드
                )
        ;
        return http.build();
    }
}
```

`CustomOidcUserService` 클래스를 아직 만들지 않아서 컴파일 에러가 발생한다. 바로 다음에 만들거니 일단 작성해주면 된다.

**`userInfoEndpoint`**   
인증된 최종 사용자에 대한 클레임(정보)을 제공하는 엔드포인트에 대한 설정을 구성한다.

**`oidcUserService`**  
OpenID Connect 1.0 프로토콜을 사용하는 인증 과정에서 사용하는 서비스를 설정한다.

`CustomOidUserService`를 만들기 전에 `OidcUser`, `UserDetails`를 구현한 `UserPrincipal`을 만들자. 이 클래스는 `CustomUserDetails` 클래스도 대신하게 만들 것이다.

```java
@Getter
public class UserPrincipal implements OidcUser, UserDetails {

    private final Long id;
    private final String email;
    private final String password;
    private final String nickname;
    private final RoleType role;
    private final OidcUser oidcUser;

    public UserPrincipal(User user, OidcUser oidcUser) {
        this.id = user.getId();
        this.email = user.getEmail();
        this.password = user.getPassword();
        this.nickname = user.getNickname();
        this.role = user.getRole();
        this.oidcUser = oidcUser;
    }

    @Override
    public Map<String, Object> getClaims() {
        return oidcUser != null ? oidcUser.getClaims() : Map.of();
    }

    @Override
    public OidcUserInfo getUserInfo() {
        return oidcUser != null ? oidcUser.getUserInfo() : null;
    }

    @Override
    public OidcIdToken getIdToken() {
        return oidcUser != null ? oidcUser.getIdToken() : null;
    }

    @Override
    public Map<String, Object> getAttributes() {
        return oidcUser != null ? oidcUser.getAttributes() : Map.of();
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Collections.singletonList(new SimpleGrantedAuthority(role.name()));
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public String getName() {
        return oidcUser != null ? oidcUser.getName() : email;
    }
}
```

`RestAuthenticationSuccessHandler`, `RestAuthenticationProvider`, `CustomUserDetailsService`에 `CustomUserDetails`를 사용하는 부분을 `UserPrincipal`로 변경해주자. 자세한 코드는 깃허브 전체 코드를 참고해주길 바란다.

이제 `CustomOidcUserService`를 만들자
```java
@Service
@RequiredArgsConstructor
public class CustomOidcUserService implements OAuth2UserService<OidcUserRequest, OidcUser> {

    private final UserRepository userRepository;

    @Override
    public OidcUser loadUser(OidcUserRequest userRequest) throws OAuth2AuthenticationException {
        OidcUserService delegate = new OidcUserService();
        OidcUser oidcUser = delegate.loadUser(userRequest);

        User user = saveOrUpdate(oidcUser, userRequest.getClientRegistration().getRegistrationId());

        return new UserPrincipal(user, oidcUser);
    }

    private User saveOrUpdate(OidcUser oidcUser, String provider) {
       User user = userRepository.findByEmail(oidcUser.getEmail())
                .map(entity -> entity.update(oidcUser.getFullName(), oidcUser.getPicture()))
                .orElseGet(() -> User.builder()
                        .deleted(false)
                        .nickname(oidcUser.getFullName())
                        .role(RoleType.ROLE_USER)
                        .provider(ProviderType.GOOGLE)
                        .email(oidcUser.getEmail())
                        .picture(oidcUser.getPicture())
                        .build());

       return userRepository.save(user);
    }
}
```

프론트 단에 `/oauth2/authorization/google`를 추가해 테스트해보면 로그인이 잘 동작하는 걸 확인할 수 있을거다.
OAuth2를 위한 로그아웃은 따로 구현하지 않아도 자체 로그인 포스팅 때 구현했던 로그아웃 엔드포인트로 로그아웃이 성공적으로 이루어진다.

# 🐡 자신의 정보 조회하기 구현
`UserController`를 만들어준다.
```java
@RestController
@RequestMapping("/api/v1/user")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping("")
    public ResponseEntity<UserResponse> getUser(@AuthenticationPrincipal UserPrincipal userPrincipal) {
        UserResponse userResponse = userService.getUser(userPrincipal);
        return ResponseEntity.ok(userResponse);
    }

}
```
<br>

`UserService`도 만들어 주자
```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    @Transactional(readOnly = true)
    public UserResponse getUser(UserPrincipal userPrincipal) {
        User findUser = userRepository.findByEmail(userPrincipal.getEmail()).orElseThrow(() -> new RuntimeException("찾을 수 없음"));
        return UserResponse.builder()
                .nickname(findUser.getNickname())
                .email(findUser.getEmail())
                .role(findUser.getRole().name())
                .build();
    }

}
```

소셜 로그인으로 정보 조회하는 것 역시 프론트 단을 따로 만들어서 테스트 해보면 잘 나오는 것을 확인할 수 있다. 자체 로그인에서 정보 조회하는 것은 포스트맨으로 테스트 해보도록 하자.

---

이것으로 Spring Security 6 버전을 사용해 소셜로그인 구현하는 기본 과정은 끝냈다.
다음 포스팅은 Naver로 소셜로그인을 구현해보도록 하자. 추후에는 Swagger를 통해 API 확인하는 것도 구현해볼 생각이다.

---

<p class="ref">참고</p>
- [스프링 시큐리티 레퍼런스](https://docs.spring.io/spring-security/reference/servlet/oauth2/index.html){: target='_blank'}