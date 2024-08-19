---
title: "[Spring Security] 인증(Authentication) 아키텍처"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/authentication-architecture/

toc: true
toc_sticky: true

date: 2024-08-19
last_modified_at: 2024-08-19
---
<blockquote class="info">이 포스팅은 스프링 시큐리티 공식문서와 스프링 시큐리티 완전 정복(인프런 강좌)을 참고해서 작성하였다.</blockquote>

---

# 👽 Authentication
- `principal`: 인증 주체를 의미하며 인증 요청의 경우 사용자 이름(아이디)을, 인증 후에는 `UserDetails` 타입의 객체가 될 수 있다.
- `credentials`: 인증 주체가 올바른 것을 증명하는 자격 증명으로서 대개 비밀번호를 의미한다. 

<!-- Authentication 은 사용자의 인증 정보를 저장하는 토큰 개념의 객체로 활용되며 인증 이후 `SecurityContext` 에 저장되어 전역적으로 참조가 가능 -->

---

# 🍕 SecurityContextHolder
![SecurityContextHolder](/assets/images/posts_img/spring-security/authentication-architecture/securitycontextholder.png)

`SecurityContextHolder` 는 현재 인증된 사용자의 정보를 [`SecurityContext`](#-securitycontext) 객체에 저장하고 있다.<!-- 스프링 시큐리티는 `SecurityContextHolder` 에 들어간 값이 어떤 과정으로 들어갔는지에는 관심이 없다. 안에 값이 있으면 현재 인증된 사용자로 간주한다.  --> 인증 주체(principal)에 대한 정보를 얻으려면 `SecurityContextHolder` 에 접근하면 된다. 아래는 `SecurityContextHolder` 를 통해 인증 정보를 조회하는 방법이다.
```java
SecurityContext context = SecurityContextHolder.getContext();
Authentication authentication = context.getAuthentication();
String username = authentication.getName();
Object principal = authentication.getPrincipal();
Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
```
<!-- 기본적으로 `SecurityContextHolder` 는 `ThreadLocal` 을 사용해 이러한 세부 정보를 저장한다. 이로 인해 `SecurityContext` 는 동일한 스레드의 메서드(methods)에서 항상 사용할 수 있으며, 해당 메서드에 `SecurityContext` 가 명시적으로 인자로 전달되지 않더라도 마찬가지다. 현재 주체의 요청이 처리된 후 스레드를 지우도록 주의를 기울인다면 이러한 방식으로 TreadLocal을 사용하는 것은 매우 안전하다. [FilterChainProxy](https://ijnooyah.github.io/spring-security/architecture/#-filterchainproxy) 는 SecurityContext 가 항상 지워지도록 보장한다. -->

기본적으로 `SecurityContextHolder` 는 `ThreadLocal` 을 사용해 보안 컨텍스트를 저장한다. `ThreadLocal` 을 사용하면 각 스레드가 독립적인 `SecurityContext` 를 가지게 되며, 동일한 스레드 내의 메서드에서 `SecurityContext` 에 접근할 수 있다.  <!-- 이는 메서드 인자로 `SecurityContext` 를 명시적으로 전달하지 않아도 되게 한다. 이 방식은 동일한 스레드 내의 메서드에서 항상 `SecurityContext` 를 사용할 수 있게 해준다. 즉, 메서드에 `SecurityContext` 가 명시적으로 인자로 전달되지 않더라도 해당 스레드에서는 `SecurityContext` 에 접근 할 수 있다. 이 접근 방식은 스레드 로컬 변수를 활용해 안전하고 효율적으로 인증 정보를 관리한다. -->  
그러나 `ThreadLocal` 은 스레드가 종료되면 자동으로 정리되지 않으므로 애플리케이션이 스레드를 재사용하거나 스레드 풀이 있는 환경에서는 주의를 기울여야 한다. [`FilterChainProxy`](https://ijnooyah.github.io/spring-security/architecture/#-filterchainproxy) 는 `SecurityContext` 가 항상 적절히 지워지도록 보장해 이러한 문제를 방지한다. 

`SecurityContextHolder` 는 `SecurityContext` 를 저장하는 다양한 전략을 제공한다. `SecurityContextHolder.MODE_THREADLOCAL` 은 기본값으로 각 스레드가 독립적인 보안 컨텍스트를 가지며 대부분의 서버 환경에 적합하다. 이외에도 `MODE_INHERITABLETHREADLOCAL`(하위 스레드가 부모 스레드의 보안 컨텍스트를 상속 받음)과 `MODE_GLOBAL`(모든 스레드가 공유하는 전역 보안 컨텍스트) 전략이 있으며 필요에 따라 선택할 수 있다. 

---

# 🍅 SecurityContext
[`SecurityContextHolder`](#-securitycontextholder) 를 통해 `SecurityContext` 를 조회할 수 있으며, 이 `SecurityContext` 는 현재 인증된 사용자의 [`Authentication`](#-authentication) 객체를 포함하고 있다.

---

# 👒 AuthenticationManager
`AuthenticationManager` 는 필터가 전달한 인증 정보를 바탕으로 인증을 수행하고, 인증 결과를 반환한다. `AuthenticationManager` 의 호출 후, 반환된 [`Authentication`](#-authentication) 객체가 [`SecurityContextHolder`](#-securitycontextholder) 에 설정된다.  
주로 사용하는 구현체는 [`ProviderManager`](#-providermanager) 다. 

<!-- "이후"는 다음과 같은 순서를 나타냅니다:

AuthenticationManager 호출: 사용자의 인증 정보를 처리하기 위해 AuthenticationManager가 호출됩니다.

인증 결과 반환: AuthenticationManager는 인증 결과를 담은 Authentication 객체를 반환합니다.

SecurityContextHolder에 설정: 반환된 Authentication 객체는 Spring Security의 필터나 컨트롤러에 의해 **SecurityContextHolder**에 설정됩니다.

즉, "이후"는 **AuthenticationManager**의 호출 후, 반환된 Authentication 객체가 **SecurityContextHolder**에 설정되는 과정을 의미합니다. 이 단계는 인증이 완료된 후에 인증 정보를 애플리케이션의 보안 컨텍스트에 저장하는 과정입니다. -->

---

# 🐝 ProviderManager
`ProviderManager` 는 [`AuthenticationManager`](#-authenticationmanager) 의 구현체다. `ProviderManager` 는 [`AuthenticationProvider`](#-authenticationprovider) 목록 중에서 인증 처리 요건에 맞는 적절한 `AuthenticationProvider` 를 찾아 인증 처리를 위임한다. `AuthenticationProvider` 인스턴스 중 아무도 인증할 수 없는 경우, `ProviderManger` 는 `ProviderNotFoundException` 을 발생시킨다.

![ProviderManager](/assets/images/posts_img/spring-security/authentication-architecture/providermanager.png)

또한 `ProviderManager` 는 선택적으로 부모 `AuthenticationManager` 를 구성할 수 있는데, 이 부모는 `AuthenticationProvider` 중 아무도 인증을 수행할 수 없는 경우에 참조된다. 부모는 모든 유형의 `AuthenticationManager` 가 될 수 있지만, 일반적으로 `ProviderManager` 의 인스턴스다.

![ProviderManager 부모](/assets/images/posts_img/spring-security/authentication-architecture/providermanager-parent.png)

<!-- 🚨🚨 캐시를 사용할 경우 고려해야할 사항이 있음 문제 생길경우 스프링 공식문서 보기-->

---

# 🍄 AuthenticationProvider
`AuthenticationProvider` 는 특정 유행의 인증을 수행하는데 사용된다. 예를 들어 `DaoAuthenticationProvider` 는 사용자 이름과 비밀번호 기반 인증을 지원하며, `JwtAuthenticationProvider` 는 JWT 토큰 인증을 지원한다. [`ProviderManager`](#-providermanager) 에는 여러 개의 `AuthenticationProvider` 인스턴스를 주입할 수 있다. 

<!-- - 사용자의 작겨 증명을 확인하고 인증 과정을 관리하는 클래스로서 사용자가 시스템에 엑세스하기 위해 제공한 정보(예: 아이디와 비밀번호)가 유효한지 검증하는 과정을 포함한다.
- 다양한 유형의 인증 메커니즘을 지원할 수 있는데, 예를 들어 표준 사용자 이름과 비밀번호를 기반으로한 인증, 토큰 기반 인증, 지문 인식 등을 처리할 수 있다.
- 성공적인 인증 후에는 Authentication 객체를 반환하며 이 객체에는 사용자의 신원 정보와 인증된 자격 증명을 포함한다.
- 인증 과정 중에 문제가 발생한 경우 AuthenticaitonExeption 과 같은 예외를 발생시켜 문제를 알리는 역할을 한다. -->

---

# 🌊 인증 흐름(Authentication Flow)
![flow](/assets/images/posts_img/spring-security/authentication-architecture/abstractauthenticationprocessingfilter.png)
1. 사용자가 자격 증명을 제출하면, `AbstractAuthenticationProcessingFilter` 는 `HttpServletRequest` 에서 [`Authentication`](#-authentication) 객체를 생성하여 인증한다. 생성되는 `Authentication` 의 유형은 `AbstractAuthenticationProcessingFilter` 의 하위 클래스에 따라 달라진다. 예를 들어, `UsernamePasswordAuthenticationFilter` 는 `HttpServletRequest` 에서 제출된 사용자 이름과 비밀번호를 바탕으로 `UsernamePasswordAuthenticationToken` 을 생성한다.
2. 그런 다음, 생성된 `Authentication` 객체를 인증을 위해 [`AuthenticationManager`](#-authenticationmanager) 에 전달한다.
3. 인증이 실패하면 다음과 같은 처리가 이루어진다.
    - [`SecurityContextHolder`](#-securitycontextholder) 가 비워진다.
    - `RememberMeServices.loginFail` 이 호출된다. "Remember Me"가 구성되지 않은 경우, 이 호출은 무시된다.
    - `AuthenticationFailureHandler` 가 호출된다.
4. 인증이 성공하면 다음과 같은 처리가 이루어진다.
    - `SessionAuthenticationStrategy` 가 새로운 로그인을 알린다.
    - `Authentication` 이 `SecurityContextHolder` 에 설정된다. 나중에 [`SecurityContext`](#-securitycontext) 를 저장해 향후 요청에서 자동으로 설정할 수 있도록 하려면 `SecurityContextRepository#saveContext` 를 명시적으로 호출해야 한다. 
    - `RememberMeServices.loginSuccess` 가 호출된다. "Remember Me"가 구성되지 않은 경우, 이 호출은 무시된다. 
    - `ApplicationEventPublisher` 가 `InteractiveAuthenticationSuccessEvent` 를 게시한다.
    - `AuthenticationSuccessHandler` 가 호출된다.

---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard)

