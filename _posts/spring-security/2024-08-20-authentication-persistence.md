---
title: "[Spring Security] 인증 영속성"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security, 스프링 시큐리티 시리즈]

permalink: /spring-security/authentication-persistence/

toc: true
toc_sticky: true

date: 2024-08-20
last_modified_at: 2024-08-29
---

{% include spring-security-list.md %}

---

# 🌞 SecurityContextRepository
![SecurityContextRepository](/assets/images/posts_img/spring-security/authentication-persistence/securitycotextrepository.png)

`SecurityContextRepository` 는 스프링 시큐리티에서 사용자가 인증을 한 이후 요청에 대해 계속 사용자의 인증을 유지하기 위해 사용되는 클래스다. 

## 🚝 HttpSessionSecurityContextRepository
`HttpSession` 에 [`SecruityContext`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontext){: target="_blank"} 를 저장한다.

## 💌 RequestAttributeSecurityContextRepository
`ServletRequest` 에 `SecurityContext` 를 저장한다. 단일 요청 내에서는 `SecurityContext` 가 유효하다. (새로운 요청시 X)

## 🏠 NullSecurityContextRepository
세션을 사용하지 않는 인증(JWT, OAuth2) 일 경우 사용하며 컨텍스트 관련 아무런 처리를 하지 않는다.

## 🐠 DelegatingSecurityContextRepository
`DelegatingSecurityContextRepository` 는 `SecurityContext` 를 여러 개의 [`SecurityContextRepository`](#-securitycontextrepository) 위임 객체에 저장하고, 지정된 순서대로 임의의 위임 객체에서 보안 컨텍스트를 가져올 수 있도록 한다.

이러한 방식으로 [`RequestAttributeSecurityContextRepository`](#-requestattributesecuritycontextrepository) 와 [`HttpSessionSecurityContextRepository`](#-httpsessionsecuritycontextrepository) 를 동시에 사용할 수 있도록 설정할 수 있다.
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
	http
		// ...
		.securityContext((securityContext) -> securityContext
			.securityContextRepository(new DelegatingSecurityContextRepository(
				new RequestAttributeSecurityContextRepository(),
				new HttpSessionSecurityContextRepository()
			))
		);
	return http.build();
}
```
스프링 시큐리티 6에서 위 예시는 기본 설정이다. (별도의 설정을 하지 않아도 `DelegatingSecurityContextRepository` 가 자동으로 구성되어 사용되고, `RequestAttributeSecurityContextRepository` 와 `HttpSessionSecurityContextRepository` 에 위임한다.)


# 👜 SecurityContextHolderFilter
`SecurityContextHolderFilter` 는 [`SecurityContextRepository`](#-securitycontextrepository) 에서 [`SecurityContext`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontext){: target="_blank"} 를 가져와 [`SecurityContextHolder`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontextholder){: target="_blank"} 에 설정하는 필터다. 

![SecurityContextHolderFilter](/assets/images/posts_img/spring-security/authentication-persistence/securitycontextholderfilter.png)

`SecurityContextPersistenceFilter` 와 달리, `SecurityContextHolderFilter` 는 `SecurityContext` 를 불러오기만 하고 저장하지 않는다. 즉, `SecurityContextHolderFilter` 를 사용할 때는 `SecurityContext` 를 **명시적으로 저장**해야 한다.

```java
/* SecurityContextPersistenceFilter 를 사용해 SecurityContextHolder 설정 */
SecurityContextHolder.setContext(securityContext);
```
위 코드는 아래와 같이 바뀌어야 한다.

```java
/* SecurityContextHolderFilter 를 사용해 SecurityContextHolder 설정 */
SecurityContextHolder.setContext(securityContext);
securityContextRepository.saveContext(securityContext, httpServletRequest, httpServletResponse); // 명시적으로 SecurityContext 저장
```

<blockquote class="info"><code>SecurityContextPersistenceFilter</code> 는 Deprecated 되었다. </blockquote>

---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/authentication/persistence.html){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}

