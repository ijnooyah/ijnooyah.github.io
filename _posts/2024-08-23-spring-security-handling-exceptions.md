---
title: "[Spring Security] 예외 처리"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/handling-exceptions/

toc: true
toc_sticky: true

date: 2024-08-23
last_modified_at: 2024-08-23
---
<blockquote class="info">이 포스팅은 스프링 시큐리티 공식문서와 스프링 시큐리티 완전 정복(인프런 강좌)을 참고해서 작성하였다.</blockquote>

---
# 🛡️ 예외 처리
`ExceptionTranslationFilter`는 보안 관련 예외를 적절한 HTTP 응답(401, 403 코드 등)으로 변환하는 역할을 한다.  
이 필터는 [`FilterChainProxy`](https://ijnooyah.github.io/spring-security/architecture/#-filterchainproxy){: target="_blank"}에 포함되어 있으며, `AuthenticationException`(인증 예외)과 `AccessDeniedException`(인가 예외)을 처리한다.

![예외 처리](/assets/images/posts_img/spring-security/handling-exceptions/exceptiontranslationfilter.png)

1. `ExceptionTranslationFilter`는 먼저 `FilterChain.doFilter(request, response)`를 호출하여 애플리케이션의 다른 부분들이 요청을 처리할 수 있도록 한다.
2. 사용자가 인증되지 않았거나 `AuthenticationException`(인증 예외)이 발생하면 *인증 프로세스를 시작한다*.
    - [`SecurityContextHolder`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontextholder){: target="_blank"}를 비운다. (현재 보안 컨텍스트 초기화)
    - `HttpServletRequest`를 저장한다. 이는 인증이 성공적으로 완료된 후 원래 요청을 다시 처리할 수 있게 하기 위함이다.
    - [`AuthenticationEntryPoint`](#-authenticationentrypoint)를 사용하여 클라이언트에게 인증을 요청한다. 보통 로그인 페이지로 리다이렉트하거나 `WWW-Authenticate` 헤더를 보낸다.
3. 반면 `AccessDeniedException`(인가 예외)이 발생하면 *접근이 거부된다*. `AccessDeniedHandler`가 호출되어 접근 거부를 처리한다.

> `AccessDeniedException` 또는 `AuthenticationException`이 발생하지 않으면, `ExceptionTranslationFilter`는 아무런 동작을 하지 않는다.

----
# 🚪 AuthenticationEntryPoint
`AuthenticationEntryPoint`는 사용자가 인증되지 않은 상태로 보호된 자원에 접근하려 할 때, 어떤 행동을 취해야 할지를 정의하는 스프링 시큐리티의 구성 요소이다.  
마치 경비원과 같이, 허가되지 않은 방문객에게 "잠시만요, 먼저 로그인해주세요."라고 안내하는 역할을 한다.

일반적으로 클라이언트는 보호된 리소스(자원)에 접근할 때 사용자이름과 비밀번호 같은 인증 정보를 요청과 함께 전송한다. 이 경우 스프링 시큐리티는 추가적인 인증 요청 없이 바로 해당 정보를 처리한다.

만약 클라이언트가 인증 정보 없이 보호된 리소스에 접근을 시도하는 경우 `AuthenticationEntryPoint`가 개입한다.  
`AuthenticationEntryPoint`의 구현체는 로그인 페이지로 리다이렉트를 수행하거나, `WWW-Authenticate` 헤더를 포함한 응답을 보내거나, 그 외 다른 조치를 취할 수 있다.

---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}

