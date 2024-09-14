---
title: "[Spring Security] Spring Security 6.x 버전 & JWT 실습 튜토리얼"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/jwt-series-intro

toc: true
toc_sticky: true

date: 2024-09-13
last_modified_at: 2024-09-14
---

[Spring Security](https://ijnooyah.github.io/spring-security/basic/), [JWT](https://ijnooyah.github.io/spring-security/jwt) 포스팅을 통해 개념에 대해 학습하였다. 이제 배운 내용을 바탕으로 직접 구현해보는 과정을 시작해보겠다. 
이제 이것들을 직접 구현해보는 과정을 시작해보자. 

---
{: .style1}

# 🚀 프로젝트 생성하기

먼저 [Spring Initializr](https://start.spring.io/)을 사용해 프로젝트를 생성해보자.

![Spring Initializr](/assets/images/posts_img/spring-security/jwt-series-intro/initializr.png)

- **의존성:** Web, JPA, H2, Lombok, Spring Security
- **Group:** com.yoonji (원하는 대로 설정)
- **Articact:** jwt (원하는 대로 설정)
- **Java 버전:** 선호하는 버전 선택


프로젝트를 생성하고 IDE를 통해 열어주면 된다.

이제 생성한 프로젝트가 정상적으로 실행되는지 확인해보자.  
프로젝트 실행을 누른 뒤

```
2024-09-13T18:55:41.713+09:00  INFO 9552 --- [jwt] [           main] r$InitializeUserDetailsManagerConfigurer : Global AuthenticationManager configured with UserDetailsService bean with name inMemoryUserDetailsManager
2024-09-13T18:55:42.749+09:00  INFO 9552 --- [jwt] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-09-13T18:55:42.789+09:00  INFO 9552 --- [jwt] [           main] com.yoonji.jwt.JwtApplication            : Started JwtApplication in 35.331 seconds (process running for 38.773)
2024-09-13T18:55:47.220+09:00  INFO 9552 --- [jwt] [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2024-09-13T18:55:47.220+09:00  INFO 9552 --- [jwt] [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2024-09-13T18:55:47.224+09:00  INFO 9552 --- [jwt] [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 4 ms
```

브라우저에서 `localhost:8080` 에 접속하면 Spring Security의 기본 로그인 페이지가 표시된다.

![동작 테스트](/assets/images/posts_img/spring-security/jwt-series-intro/localhost8080.png)

사진과 같은 화면이 뜨면 잘 동작하고 있는 거다.

---
{: .style1}

# 🗃️ 데이터베이스 설정하기 (H2)

![h2](/assets/images/posts_img/spring-security/jwt-series-intro/h2-version.png)

위 사진과 같이 프로젝트에서 H2 버전을 확인해주고 [https://www.h2database.com](https://www.h2database.com)로 들어가서 해당 버전을 설치해주면 된다.

H2를 실행하는 방법은 참고자료가 많으므로 생략하겠다.

JDBC URL 부분을
처음에는 `jdbc:h2:~/jwt`로 설정한 후 연결해준다. 
그리고 `~/jwt.mv.db` 파일이 생성 됐는지 확인해 준다.
윈도우에서는 로컬디스크 > 사용자 > 사용자명 폴더에 들어가보면 된다.

![생성 파일](/assets/images/posts_img/spring-security/jwt-series-intro/h2-file.png)
생성된 jwt.mv.db 파일

이후로는 `jdbc:h2:tcp://localhost/~/jwt`로 접속해주면 된다.

다음으로 `application.yml` 파일에 DB 정보를 입력해주자.

기존에 있던 `application.properties` 파일은 삭제해주고 `application.yml`을 생성해준다.

**appliction.yml**
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

logging.level:
  org.hibernate.SQL: debug
  org.springframework.security: debug
```

<br>

DB 연동을 확인하기 위해 간단한 `User` 엔티티와 리포지토리를 생성하고 테스트를 해보자.
<br>

**회원 엔티티**
```java
@Entity
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    private Long id;

    private String name;
    private String email;

    public User (String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

<br>

**회원 리포지토리**
```java
public interface UserRepository extends JpaRepository<User, Long> {
}
```

<br>

**회원 테스트** 
```java
@SpringBootTest
@Transactional
class UserTest {

    @Autowired
    UserRepository userRepository;

    @Test
    public void testMember() throws Exception {
        User user = new User("하윤지", "ijnooyah@gmail.com");
        User savedUser = userRepository.save(user);

        User findUser = userRepository.findById(savedUser.getId())
                .orElseThrow(() -> new IllegalArgumentException("Member not found with id: " + savedUser.getId()));

        assertThat(findUser.getId()).isEqualTo(user.getId());
        assertThat(findUser.getName()).isEqualTo(user.getName());
        assertThat(findUser.getEmail()).isEqualTo(user.getEmail());

        assertThat(findUser).isEqualTo(user);
    }
}
```

<br>

테스트가 통과하면 DB 연동까지 완료된 것이다.
이제 이 프로젝트에서 사용할 도메인 모델을 알아보자.

---
{: .style1}

# 📝 엔티티 작성하기

JPA를 공부하는 것이 아니므로 최대한 간단한 도메인 모델을 설계했다.

- 회원(User)과 리프레시토큰(RefreshToken): 일대일 관계

이제 각 엔티티에 대한 코드를 작성해보자.

그전에 스프링 부트 애플리케이션 클래스에 `@EnableJpaAuditing` 어노테이션을 추가해주자.

**JwtApplication.java** 
```java
@SpringBootApplication
@EnableJpaAuditing
public class JwtApplication {

	public static void main(String[] args) {
		SpringApplication.run(JwtApplication.class, args);
	}

}
```

<br>

**BaseTimeEntity.java**
```java
@Getter
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class BaseTimeEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

<br>

**User.java**
```java
@Entity
@Table(name = "users")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class User extends BaseTimeEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_id")
    private Long id;

    private String name;
    private String email;
    private String password;

    @Enumerated(EnumType.STRING)
    private RoleType role;

    private boolean deleted;

    private LocalDateTime deletedAt;

    @Builder
    public User (String name, String email, String password, RoleType role, boolean deleted, PasswordEncoder passwordEncoder) {
        this.name = name;
        this.email = email;
        this.encodePassword(password, passwordEncoder);
        this.role = role;
        this.deleted = deleted;
    }

    private void encodePassword(String rawPassword, PasswordEncoder passwordEncoder) {
        if (rawPassword != null && !rawPassword.isEmpty()) {
            this.password = passwordEncoder.encode(rawPassword);
        }
    }
}
```

<br>

**RoleType.java**
```java
public enum RoleType {
    ROLE_USER, ROLE_GUEST
}
```

<br>

**RefreshToken.java**
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class RefreshToken extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "refresh_token_id")
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    private String token;

    private LocalDateTime expireDate;

    private boolean revoked;

    @Builder
    public RefreshToken(User user, String token, LocalDateTime expireDate, boolean revoked) {
        this.user = user;
        this.token = token;
        this.expireDate = expireDate;
        this.revoked = revoked;
    }
}
```
<br>


---
{: .style1}


이것으로 기본적인 프로젝트 설정이 완료 되었다. [다음 포스팅](https://ijnooyah.github.io/spring-security/implementig-jwt)에서 JWT를 이용해 로그인, 로그아웃, 사용자 정보 조회, 토큰 재발급 기능을 구현해보자.









