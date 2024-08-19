---
title: "[Spring Security] Spring Security 아키텍처"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/architecture/

toc: true
toc_sticky: true

date: 2024-08-17
last_modified_at: 2024-08-19
---
<blockquote class="info">이 포스팅은 스프링 시큐리티 공식문서와 스프링 시큐리티 완전 정복(인프런 강좌)을 참고해서 작성하였다.</blockquote>

<blockquote>Spring Security’s Servlet support is based on Servlet Filters, so it is helpful to look at the role of Filters generally first.
<br>
<em>- 스프링 시큐리티 공식 문서 -</em>
</blockquote>

공식 문서에서 스프링 시큐리티는 서블릿 필터가 바탕이라고 한다. 따라서 [필터](https://ijnooyah.github.io/spring/filter/)를 먼저 알고 가는 것이 도움이 된다.  

참고로 스프링 시큐리티는 별도의 설정을 하지 않아도 스프링 시큐리티의 기본적인 웹 보안 기능이 작동한다.

<!-- ---

# SecurityBuilder
- 빌더 클래스
- 웹 보안을 구성하는 빈(Bean) 객체와 설정 클래스 생성
- [HttpSecurity](#httpsecurity), [WebSecurity](#websecurity) 가 있음

---

# SecurityConfigurer
![security-builder-secruity-configurer](/assets/images/posts_img/spring-security/architecture/security-builder-secruity-configurer.png)
- 여러 초기화 설정에 관여
- HTTP 요청과 관련된 보안처리를 담당하는 필터들 생성 -->

---
# 🐵 HttpSecurity
![HttpSecurity](/assets/images/posts_img/spring-security/architecture/HttpSecurity.png)
`HttpSecurityConfiguration` 에서 `HttpSecurity` 를 생성하고 초기화를 진행한다. `HttpSecurity` 는 보안에 필요한 각 설정 클래스와 필터들을 생성하고 **최종적으로 [`SecurityFilterChain`](#-securityfilterchain) 빈 생성**

---

# 🐶 WebSecurity
![WebSecurity](/assets/images/posts_img/spring-security/architecture/WebSecurity.png)
`WebSecurityConfiguration` 에서 `WebSecurity` 를 생성하고 초기화를 진행한다. [`HttpSecurity`](#-httpsecurity) 에서 생성한 [`SecurityFilterChain`](#-securityfilterchain) 빈을 `SecurityBuilder`에 저장한다. `build()` 를 실행하면 `SecurityBuilder` 에서 `SecurityFilterChain` 을 꺼내어 [`FilterChainProxy`](#-filterchainproxy) 생성자에게 전달한다. (`FilterChainProxy` 생성)

<!-- ![debug](/assets/images/posts_img/spring-security/architecture/WebSecurity-debug.png) -->

---

# 🐷 DelegatingFilterProxy
`DelegatingFilterProxy` 은 스프링에서 사용되는 특별한 서블릿 필터로 **서블릿 컨테이너와 스프링 애플리케이션 컨텍스트 간의 연결고리** 역할을 한다. [`FilterChainProxy`](#-filterchainproxy) 에게 요청을 **위임**한다. (실제 보안 처리 수행 X)

---
# 🐭 FilterChainProxy
`FilterChainProxy` 은 [`DelelatingFilterProxy`](#-delegatingfilterproxy) 로부터 요청을 위임 받는다. <!--받고 보안 처리 역할을 함 -->
내부적으로 하나 이상의 [`SecurityFilterChain`](#-securityfilterchain) 객체들을 가지고 있으며 요청 URL 정보를 기준으로 적절한 `SecurityFilterChain` 을 선택하여 필터들을 호출한다.
![FilterChainProxy](/assets/images/posts_img/spring-security/architecture/multi-securityfilterchain.png)

---
# 🦊 SecurityFilterChain
`SecurityFilterChain` 은 [`FilterChainProxy`](#-filterchainproxy)에 의해 사용되며, 현재 요청에 대해 어떤 스프링 시큐리티 `Filter` 가 호출되어야 하는지 결정한다.
- `List<Filter> getFilters()`: 현재 `SecurityFilterChain`에 포함된 `Filter` 객체의 리스트 반환
- `boolean matches(HttpServletRequest request)`: 요청이 현재 `SecurityFilterChain`에 의해 처리되어야 하는지 여부를 결정 

---
# 🐻 사용자 정의 보안 기능 구현
```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {
  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
      .formLogin(Customizer.withDefaults());
  return http.build();
  }
}
```
- `@EnableWebSecurity`을 클래스에 정의한다.
- 모든 설정 코드는 람다 형식으로 작성한다. (스프링 시큐리티 7버전 부터는 람다 형식만 지원할 예정)
- [`SecurityFilterChain`](#-securityfilterchain) 을 빈으로 정의하게 되면 자동설정에 의한 `SecurityFilterChain` 빈은 생성되지 않는다.

---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard)

