---
title: "[Spring Security] Spring Security 6.x 버전 OAuth2 & JWT 실습 튜토리얼"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/oauth2-jwt-series-intro

toc: true
toc_sticky: true

date: 2024-09-11
last_modified_at: 2024-09-11

published: false
---

[Spring Security](https://ijnooyah.github.io/spring-security/basic/), [JWT](https://ijnooyah.github.io/spring-security/jwt), 그리고 [OAuth2](https://ijnooyah.github.io/spring-security/oauth2) 포스팅을 통해 개념에 대해 학습하였다. 이제 배운 내용을 바탕으로 직접 구현해보는 과정을 시작해보겠다. 
이제 이것들을 직접 구현해보는 과정을 시작해보자.  

---
{: .style1}

# 🚀 프로젝트 생성하기

먼저 [Spring Initializr](https://start.spring.io/)을 사용해 프로젝트를 생성해보자.

![Spring Initializr](/assets/images/posts_img/spring-security/oauth2-jwt-series-intro/initializr.png)

- **의존성:** Web, JPA, H2, Lombok, Spring Security
- **Group:** com.yoonji (원하는 대로 설정)
- **Articact:** demo (원하는 대로 설정)
- **Java 버전:** 선호하는 버전 선택


프로젝트를 생성하고 IDE를 통해 열어주면 된다.

이제 생성한 프로젝트가 정상적으로 실행되는지 확인해보자.  
프로젝트 실행을 누른 뒤

```
INFO 4688 --- [security] [           main] r$InitializeUserDetailsManagerConfigurer : Global AuthenticationManager configured with UserDetailsService bean with name inMemoryUserDetailsManager
2024-09-11T09:19:08.589+09:00  INFO 4688 --- [security] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2024-09-11T09:19:08.609+09:00  INFO 4688 --- [security] [           main] c.yoonji.security.SecurityApplication    : Started SecurityApplication in 12.912 seconds (process running for 18.784)
```

브라우저에서 `localhost:8080` 에 접속하면 Spring Security의 기본 로그인 페이지가 표시된다.

![동작 테스트](/assets/images/posts_img/spring-security/oauth2-jwt-series-intro/localhost8080.png)

사진과 같은 화면이 뜨면 잘 동작하고 있는 거다.

---
{: .style1}

# 🗃️ 데이터베이스 설정하기 (H2)

![h2](/assets/images/posts_img/spring-security/oauth2-jwt-series-intro/h2-version.png)

위 사진과 같이 프로젝트에서 H2 버전을 확인해주고 [https://www.h2database.com](https://www.h2database.com)로 들어가서 해당 버전을 설치해주면 된다.

H2를 실행하는 방법은 참고자료가 많으므로 생략하겠다.

![h2 화면](/assets/images/posts_img/spring-security/oauth2-jwt-series-intro/h2.png)

JDBC URL 부분을
처음에는 `jdbc:h2:~/security`로 설정한 후 연결해준다. 
그리고 `~/security.mv.db` 파일이 생성 됐는지 확인해 준다.
윈도우에서는 로컬디스크 > 사용자 > 사용자명 폴더에 들어가보면 된다.

![생성 파일](/assets/images/posts_img/spring-security/oauth2-jwt-series-intro/h2-file.png)
생성된 security.mv.db 파일

이후로는 `jdbc:h2:tcp://localhost/~/security`로 접속해주면 된다.

다음으로 `application.yml` 파일에 DB 정보를 입력해주자.

기존에 있던 `application.properties` 파일은 삭제해주고 `application.yml`을 생성해준다.

**appliction.yml**
```yml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/security
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
```

<br>

DB 연동을 확인하기 위해 간단한 `Member` 엔티티와 리포지토리를 생성하고 테스트를 해보자.
<br>

**회원 엔티티**
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String loginId;
    private String username;
    private String email;

    public Member(String loginId, String username, String email) {
        this.loginId = loginId;
        this.username = username;
        this.email = email;
    }

}
```

<br>

**회원 리포지토리**
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}

```

<br>

**회원 테스트** 
```java
@SpringBootTest
@Transactional
class MemberTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    public void testMember() throws Exception {
        Member member = new Member("ijnooyah", "하윤지", "ijnooyah@gmail.com");
        Member savedMember = memberRepository.save(member);

        Member findMember = memberRepository.findById(savedMember.getId())
                .orElseThrow(() -> new IllegalArgumentException("Member not found with id: " + savedMember.getId()));

        assertThat(findMember.getId()).isEqualTo(member.getId());
        assertThat(findMember.getLoginId()).isEqualTo(member.getLoginId());
        assertThat(findMember.getUsername()).isEqualTo(member.getUsername());
        assertThat(findMember.getEmail()).isEqualTo(member.getEmail());

        assertThat(findMember).isEqualTo(member);
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

- 회원(Member)과 리프레시토큰(RefreshToken): 일대일 관계

이제 각 엔티티에 대한 코드를 작성해보자.

**BaseTimeEntity.java**
```java
@Getter
@EntityListeners(AuditingEntityListener.class) // 스프링 부트 애플리케이션 클래스에 @EnableJpaAuditing 추가
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

**Member.java**
```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id")
    private Long id;

    private String email;
    private String password;
    private String name;


    @Enumerated(EnumType.STRING)
    private ProviderType provider; // 외부 인증 제공자의 이름

    private String providerId; // 외부 인증 제공자에게 부여받은 고유 식별자

    @Enumerated(EnumType.STRING)
    private RoleType role;

    @OneToOne(mappedBy = "member", fetch = FetchType.LAZY)
    private RefreshToken refreshToken;

    @Builder
    public Member(String name, String email, ProviderType provider, String providerId, RoleType role) {
        this.name = name;
        this.email = email;
        this.provider = provider;
        this.providerId = providerId;
        this.role = role;
    }

    public Member update(String name) {
        this.name = name;
        return this;
    }

    public void changeRefreshToken(RefreshToken refreshToken) {
        this.refreshToken = refreshToken;
        refreshToken.changeMember(this);
    }
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
    @JoinColumn(name = "member_id")
    private Member member;

    private String token;

    private LocalDateTime expireDate;

    public void changeMember(Member member) {
        this.member = member;
    }
}
```
<br>

**ProviderType.java**
```java
public enum ProviderType {
    LOCAL, GOOGLE
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

---
{: .style1}

# 🔌 의존성 추가하기

```gradle 
implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
implementation 'org.springframework.boot:spring-boot-starter-oauth2-client'
```
`build.gradle`에 위의 의존성들을 추가해준다.

`spring-security-oauth2-resource-server` 의존성은 OAuth2 리소스 서버 기능을 제공한다. JWT 검증, 토큰 기반 인증 등의 기능이 있다.

`spring-security-oauth2-client` 의존성은 OAuth2 클라이언트 기능을 제공한다. OAuth2 로그인, 토큰 관리 등의 기능이 있다.


이것으로 기본적인 프로젝트 설정이 완료 되었다. 다음 포스팅에서는 OAuth2 클라이언트 구현을 해보겠다.









