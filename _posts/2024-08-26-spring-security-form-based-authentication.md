---
title: "[Spring Security] 폼 기반 인증"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/form-based-authentication/

toc: true
toc_sticky: true

date: 2024-08-26
last_modified_at: 2024-08-26
---
<blockquote class="info">이 포스팅은 스프링 시큐리티 공식문서와 스프링 시큐리티 완전 정복(인프런 강좌)을 참고해서 작성하였다.</blockquote>

---
# 🌊 폼 기반 인증 흐름
스프링 시큐리티에서 폼 기반 인증은 주로 `UsernamePasswordAuthenticationFilter`를 통해 처리된다. `UsernamePasswordAuthenticationFilter`는 `AbstractAuthenticationProcessingFilter`를 상속받아 구현되었으며, [기본적인 인증 흐름](https://ijnooyah.github.io/spring-security/authentication-architecture/#-%EC%9D%B8%EC%A6%9D-%ED%9D%90%EB%A6%84authentication-flow)을 따른다.

![Authenticating Username and Password](/assets/images/posts_img/spring-security/form-based-authentication/usernamepasswordauthenticationfilter.png)사용자이름과 비밀번호를 이용한 인증 과정

**1.** 사용자가 로그인 정보를 제출하면, `UsernamePasswordAuthenticationFilter`가 `HttpServletRequest`에서 사용자이름과 비밀번호를 추출해 `UsernamePasswordAuthenticationToken`([`Authentication`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-authentication)의 한 종류)을 생성한다.  

**2.** 생성된 `UsernamePasswordAuthenticationToken`은 [`AuthenticationManager`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-authenticationmanager)에 전달되어 실제 인증 작업이 이루어진다.   

**3.** 인증이 실패 시:
1. [`SecurityContextHolder`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontextholder)의 내용이 지워진다.
2. `RememberMeServices.loginFail`이 호출된다.(Remember me 기능이 설정된 경우에만 동작)
3. `AuthenticationFailureHandler`가 호출된다.  

**4.** 인증이 성공 시:
1. `SessionAuthenticationStrategy`에 새로운 로그인 정보가 전달된다.
2. `SecurityContextHolder`에 `Authentication`이 설정된다.
3. `RememberMeServices.loginSuccess`가 호출된다.(Remember me 기능이 설정된 경우에만 동작)
4. `ApplicationEventPublisher`가 `InteractiveAuthenticationSuccessEvent`를 발행한다.
5. `AuthenticationSuccessHandler`가 호출된다. 보통 이 핸들러는 `SimpleUrlAuthenticationSuccessHandler`의 인스턴스로, 로그인 전에 사용자가 접근하려 했던 페이지로 리다이렉트 한다.

---

# 🧑‍💻 커스텀 설정
스프링 시큐리티는 사용자 이름과 비밀번호를 통한 폼 기반 인증을 기본적으로 지원한다. 이 방식은 별도의 설정 없이도 활성화되어 있으며, 기본 로그인 페이지를 제공한다.  

그러나 서블릿 기반의 설정을 직접 제공할 경우(`SecurityFilterChain`을 `@Bean`으로 정의) 폼 기반 로그인을 명시적으로 설정해야 한다.
```java
public SecurityFilterChain filterChain(HttpSecurity http) {
	http
		.formLogin(withDefaults());
	// ...
}
```
대부분의 애플리케이션은 커스텀 로그인 폼을 요구하는데, 아래는 커스텀 로그인 폼을 설정하는 방법을 보여준다.
```java
public SecurityFilterChain filterChain(HttpSecurity http) {
	http
		.formLogin(form -> form
			.loginPage("/login")
			.permitAll()
		);
	// ...
}
```

  



---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard)

