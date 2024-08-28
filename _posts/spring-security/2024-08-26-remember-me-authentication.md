---
title: "[Spring Security] 기억하기 인증"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/remember-me-authentication/

toc: true
toc_sticky: true

date: 2024-08-26
last_modified_at: 2024-08-26
---
<blockquote class="info">이 포스팅은 스프링 시큐리티 공식문서와 스프링 시큐리티 완전 정복(인프런 강좌)을 참고해서 작성하였다.</blockquote>

---
# 💾  Remember Me 인증
사용자가 웹 사이트나 애플리케이션에 로그인할 때 자동으로 인증 정보를 기억하는 기능이다.  
[`UsernamePasswordAuthenticationFilter`](https://ijnooyah.github.io/spring-security/form-based-authentication/#-%ED%8F%BC-%EA%B8%B0%EB%B0%98-%EC%9D%B8%EC%A6%9D-%ED%9D%90%EB%A6%84){: target="_blank"}와 함께 사용되며, `AbsractAuthenticationProcessingFilter` 슈퍼클래스에서 훅을 통해 구현된다.
- 인증 성공 시: `RememberMeServices.loginSuccess()`를 통해 RememberMe 토큰을 생성하고 쿠키로 전달한다.
- 인증 실패 시: `RememberMeServices.loginFail()`를 통해 쿠키를 지운다.
- `LogoutFilter`와 연계해서 로그아웃 시 쿠키를 지운다.

><b>❓ 훅을 통해 구현된다</b>  
"훅을 통해 구현된다"라는 표현은 '훅 메서드'패턴을 의미한다.  
훅(Hook)이란 기본 동작(이미 정의된)을 변경하거나 확장할 수 있는 지점(특정 위치)을 말한다.  
훅 메서드 패턴은 상위 클래스에서 미리 정의된 메서드를 하위 클래스에서 오버라이드하여 특정 동작을 커스터마이징하는 설계 패턴이다.  
즉, Remember Me 기능이 인증 처리의 주요 지점에서 동작할 수 있도록, 미리 정의된 메서드(훅)를 통해 구현된다는 의미이다. 이는 코드의 재사용성과 확장성을 높이는 설계 방식이다.   
`AbstractAuthenticationProcessingFilter`는 `successfulAuthentication()`과 `unsuccessfulAuthentication()` 메서드를 훅 메서드로 제공한다.

---

# 🕰️ RememberMeAuthenticationFilter
[`SecurityContextHolder`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontextholder)에 `Authentication`이 포함되지 않은 경우 실행되는 필터이다.

![RememberMeAuthenticationFilter](/assets/images/posts_img/spring-security/remember-me-authentication/remembermeauthenticationfilter.png)


<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}

