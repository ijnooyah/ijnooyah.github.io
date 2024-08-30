---
title: "[Spring Security] 요청 수준 인가 & 메소드 수준 인가"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security]

permalink: /spring-security/request-and-method-authorization/

toc: true
toc_sticky: true

date: 2024-08-29
last_modified_at: 2024-08-29
---

{% include spring-security-list.md %}

---
# 📚 개요
스프링 시큐리티는 요청 수준 인가(Request-level Authorization)와 메소드 수준 인가(Method-level Authorization)을 통해 자원에 대한 심층적인 방어를 제공한다.

<b>📄 목차</b>
- [요청 수준 인가](#️-요청-수준-인가)
  - [요청 수준 인가 작동 방식](#-요청-수준-인가-작동-방식)
  - [authorizeHttpRequests()](#-authorizehttprequests)
  - [securityMatcher()](#-securitymatcher)
  - [정적 자원 관리](#️-정적-자원-관리)
- [메서드 수준 인가](#-메서드-수준-인가)
  - [메서드 수준 인가 작동 방식](#️-메서드-수준-인가-작동-방식)
  - [@PreAuthorize를 사용한 메서드 호출 인가](#-preauthorize를-사용한-메서드-호출-인가)
  - [@PostAuthorize를 사용한 메서드 결과 인가](#️-postauthorize를-사용한-메서드-결과-인가)
  - [@PreFilter를 사용한 메서드 매개변수 필터링](#-prefilter를-사용한-메서드-매개변수-필터링)
  - [@PostFilter를 메서드 결과 필터링](#-postfilter를-메서드-결과-필터링)
- [요청 수준 인가 VS 메서드 수준 인가](#-요청-수준-인가-vs-메서드-수준-인가)

---

# 🛡️ 요청 수준 인가
요청 수준 인가는 클라이언트의 HTTP 요청, 즉 `HttpServletRequest`에 대한 인가를 처리하는 방식이다. 이를 위해 [`HttpSecurity`](https://ijnooyah.github.io/spring-security/architecture/#-httpsecurity) 인스턴스를 사용해 요청에 대한 접근 제어 규칙을 선언할 수 있다.

기본적으로 Spring Securitry는 모든 요청이 인증을 요구하도록 설정되어있다.  
따라서 `HttpSecurity` 인스턴스를 사용할 때는 인가 규칙을 명시적으로 선언해야 한다. 
```java
http
    .authorizeHttpRequests((authorize) -> authorize
        .anyRequest().authenticated()
    )
```
위 코드는 가장 기본적인 인가 규칙을 설정하는 예시로, 모든 엔드포인트(`anyRequest()`)가 최소한 보안 컨텍스트에서 인증되어야(`authenticated()`) 요청을 허용한다고 알려준다.

## 🔍 요청 수준 인가 작동 방식

![Authorize HttpServletRequest](/assets/images/posts_img/spring-security/request-and-method-authorization/authorizationfilter.png)

- **1.** `AuthorizationFilter`는 `Supplier`를 생성해 [`SecurityContextHolder`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-securitycontextholder)에서 [`Authentication`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-authentication)을 검색한다.
- **2.** `Supplier<Authentication>`와 `HttpServletRequest`를 [`AuthorizationManager`](https://ijnooyah.github.io/spring-security/authorization-architecture/#-authorizationmanager)에 전달한다. `AuthorizationManager`는 요청을 [`authorizeHttpRequests()`](#-authorizehttprequests)의 패턴에 매칭하고 해당 인가 규칙을 실행한다. 
  - **3.** 인가가 거부되면, `AuthorizationDeniedEvent`가 게시되고, `AccessDeniedException`이 발생한다. 이 경우 [`ExceptionTranslationFilter`](https://ijnooyah.github.io/spring-security/handling-exceptions/#%EF%B8%8F-%EC%98%88%EC%99%B8-%EC%B2%98%EB%A6%AC)가 `AccessDeniedException`을 처리한다.
  - **4.** 접근이 허용되면, `AuthorizationGrantedEvent`가 게시되고, `AuthorizationFilter`는 `FilterChain`을 계속 진행하여 애플리케이션이 정상적으로 처리된다.

<b>✔️ 기본적으로 AuthorizationFilter는 마지막이다</b>  
`AuthorizationFilter`는 기본적으로 스프링 시큐리티 필터 체인에서 마지막으로 위치한다. 이는 스프링 시큐리티의 인증 필터, 악용 방지 및 다른 필터들이 접근 권환 확인(인가)을 요구하지 않는다는 것을 의미한다.
즉, `AuthorizationFilter` 이전에 처리되는 부분들은 접근 제어 로직의 영향을 받지 않는다는 말이고 이후에 오는 것들(DispatcherServlet 등)은 접근 제어 로직의 영향을 받아 엔드포인트가 `authorizeHttpRequests()`에 포함될 필요가 있다.

<b>✔️ Authentication 조회는 지연된다</b>  
[`AuthorizationManager`](https://ijnooyah.github.io/spring-security/authorization-architecture/#-authorizationmanager) API는 `Supplier<Authentication>`을 사용한다. 이는 요청이 항상 허용되거나 항상 거부되는 경우 인증을 조회하지 않으며, 이는 요청 속도를 높인다.

<b>✔️ 모든 디스패치는 인가 처리된다</b>  
`AuthorizationFilter`는 요청뿐만 아니라 모든 디스패치에서 실행된다. 이는 `REQUEST` 디스패치뿐만 아니라 `FORWARD`, `ERROR` 및 `INCLUDE` 디스패치에서도 권한 부여가 필요하다는 것을 의미한다. 자세한 내용은 [문서](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html#_all_dispatches_are_authorized)를 참고하자.

---

## 🚦 authorizeHttpRequests()
`authorizeHttpRequests()`는 HTTP 요청을 기반으로 접근을 제한할 수 있다.  
사용자에게 허용할 요청 엔드포인트, 필요한 권한, HTTP 메서드 등을 설정할 때 사용한다.  
`authorizeHttpRequests()`을 통해 요청과 권한 규칙이 설정되면 내부적으로 `AuthorizationFilter`가 요청에 대한 권한 검사 및 승인 작업을 수행한다

<b>🎯 requestMathcers()</b>  
- `requestMatchers()`는 HTTP 요청의 URL 패턴, HTTP 메소드, 요청 파라미터 등을 기반으로 어떤 요청에 대해서는 특정 보안 설정을 적용하고 다른 요청에 대해서는 적용하
지 않도록 세밀하게 제어할 수 있게 해 준다.
- 예를 들어 특정 API 경로에만 CSRF 보호를 적용하거나, 특정 경로에 대해 인증을 요구하지 않도록 설정할 수 있다. 이를 통해 애플리케이션의 보안 요구 사항에 맞춰서 유연한 보안 정책을 구성할 수 있다.

```java
http.authorizeHttpRequests(authorize -> authorize
.requestMatchers("/user"). hasRole("USER") //엔드 포인트와 권한 설정, 요청이 /user 엔드포인트 요청인 경우 USER 권한을 필요로 한다
.requestMatchers( "/mypage/**").hasRole("USER") //Ant 패턴을 사용할 수 있다. 요청이 /mypage 또는 하위 경로인 경우 USER 권한을 필요로 한다
.requestMatchers(RegexRequestMatcher.regexMatcher("/resource/[A-Za-z0-9]+")).hasRole("USER") //정규 표현식을 사용할 수 있다
.requestMatchers(HttpMethod.GET, "/**").hasRole("read") //HTTP METHOD 를 옵션으로 설정할 수 있다
.requestMatchers(HttpMethod.POST).hasRole("write") // POST 방식의 모든 엔드포인트 요청은 write 권한을 필요로 한다
.requestMatchers(new AntPathRequestMatcher("/manager/**")).hasAuthority("MANAGER") // 원하는 RequestMatcher 를 직접 사용할 수 있다
.requestMatchers("/admin/**").hasAnyRole("ADMIN","MANAGER") // /admin/ 이하의 모든 요청은 ADMIN 또는 MANAGER 권한을 필요로 한다
.anyRequest().authenticated()); // 위에서 정의한 규칙 외 모든 엔드포인트 요청은 인증을 필요로 한다(인증만 되면 접근할 수 있다)
```
<b>🚨 주의사항!</b>
- 스프링 시큐리티는 클라이언트의 요청에 대하여 위에서 부터 아래로 나열된 순서대로 처리하며 요청에 대하여 첫 번째 일치만 적용되고 다음 순서로 넘어가지 않는다.
- `/admin/**` 가 `/admin/db` 요청을 포함하므로 의도한 대로 권한 규칙이 올바르게 적용 되지 않을 수 있다. 그렇기 때문에 엔드 포인트 설정 시 좁은 범위의 경로를 먼저 정의하고 그것 보다 큰 범위의 경로를 다음 설정으로 정의 해야 한다.

<b>✔️ 권한 규칙 종류</b>  

| 종류 | 특징 |
| --- | --- |
| **authenticated** | 인증된 사용자만 접근할 수 있다. |
| **permitAll** | 요청에 권한이 필요하지 않으며 공개된 엔드포인드이다. [`Authentication`](https://ijnooyah.github.io/spring-security/authentication-architecture/#-authentication)은 세션에서 검색되지 않는다. |
| **denyAll** | 요청이 어떠한 상황에서도 허용되지 않는다. `Authentication`은 세션에서 검색되지 않는다. |
| **hasAuthority** | 사용자의 `Authentication`에는 주어진 값(인자)과 일치하는 GrantedAuthority가 있어야한다. |
| **hasRole** | `hasAuthority`의 단축키로, 기본적으로 `ROLE_` 접두사를 추가해 권한을 확인한다. (인자에 `ROLE_`을 제외해야한다.) |
| **hasAnyAuthority** | 사용자의 `Authentication`에는 주어진 값들(인자) 중 하나라도 일치하는 GrantedAuthority가 있어야한다.|
| **hasAnyRole** | `hasAnyAuthority`의 단축키로, 기본적으로 `ROLE_` 접두사를 추가해 권한을 확인한다. (인자에 `ROLE_`을 제외해야한다.) |

권한 규칙은 스프링 시큐리티 설정에서 정의되며, [`AuthorizationManager`](https://ijnooyah.github.io/spring-security/authorization-architecture/#-authorizationmanager)는 이러한 규칙을 기반으로 요청에 대한 접근을 평가하고 허용 또는 거부를 결정한다. 복잡한 시스템에서는 여러 `AuthorizationManager`를 사용하여 다양한 보안 요구사항을 처리할 수 있다.

---

## 🌐 securityMatcher()
특정 요청에 해당할 때만 [`HttpSecurity`](https://ijnooyah.github.io/spring-security/architecture/#-httpsecurity)가 호출되도록 구성할 수 있다.
즉, 특정 요청 엔드포인트에만 보안 필터를 적용할지를 결정할 수 있다. 

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

	@Bean
	public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
		http
			.securityMatcher("/api/**")                            
			.authorizeHttpRequests(authorize -> authorize
				.requestMatchers("/user/**").hasRole("USER")       
				.requestMatchers("/admin/**").hasRole("ADMIN")     
				.anyRequest().authenticated()                      
			)
			.formLogin(withDefaults());
		return http.build();
	}
}
```
위 설정에서는 `/api/`로 시작하는 URL에만 이 특정 [`SecurityFilterChain`](https://ijnooyah.github.io/spring-security/architecture/#-securityfilterchain)의 보안 규칙이 적용된다.  
다른 URL들은 이 특정 `SecurityFilterChain`의 영향을 받지 않지만, 다른 보안 설정이나 스프링 시큐리티의 기본 보안 설정의 영향을 받을 수 있다.

<div class='mermaid'>
  graph LR
    A[HTTP 요청] --> B{"/api/**" 매칭?}
    B -->|Yes| C["/api/users"]
    B -->|Yes| D["/api/other"]
    B -->|No| E["/admin/api/dashboard"]
    B -->|No| F["/users"]
    
    C --> G[SecurityFilterChain 적용]
    D --> G
    E --> H[SecurityFilterChain 미적용]
    F --> H
</div>

---
## 🖼️ 정적 자원 관리
```java
http
    .authorizeHttpRequests((authorize) -> authorize
        .requestMatchers("/css/**").permitAll()
        .anyRequest().authenticated()
    )
```
<b>`ignoring`보다 <u>`permitAll` 권장</u></b>
- 정적 리소스라 하더라도 보안 헤더를 작성하는 것은 중요하다. 요청이 무시되면 스프링 시큐리티가 처리할 수 없다.
- 이전에는 모든 요청마다 세션을 확인해야 했기 때문에 성능 저하가 있었지만, 스프링 시큐리티 6부터는 인가 규칙에 필요할 때만 세션을 확인한다.
- 성능 문제가 해결됐기 때문에, 스프링 시큐리티는 모든 요청에 대해 최소한 `permitAll`을 사용할 것을 권장한다.

---

# 🧩 메서드 수준 인가
메서드 수준 인가는 특정 메소드 호출에 대한 인가를 처리하는 방식이다. 서비스나 컨트롤러 메서드에 권한을 선언적으로 적용할 수 있으며, 주로 `@PreAuthorize`, `@PostAuthorize`와 같은 어노테이션을 사용하여 권한 규칙을 선언한다.

메서드 수준 인가를 활성화하기 위해서는 `@Configuration` 클래스에 `@EnableMethodSecurity`를 추가해야 한다.

메서드 수준 인가는 다음과 같은 경우에 유용하다:
- 세분화된 인가 로직을 추출; 예: 메서드 매개변수 및 리턴값이 인가 결정에 기여할 때
- 서비스 레이어에서 보안 적용
- `HttpSecurity` 기반 설정보다 어노테이션 기반 선호

---

## 🛠️ 메서드 수준 인가 작동 방식
메서드 수준 인가는 메서드 실행 전 인가와 메서드 실행 후 인가의 조합이다.
```java
@Service
public class MyCustomerService {
    @PreAuthorize("hasAuthority('permission:read')")
    @PostAuthorize("returnObject.owner == authentication.name")
    public Customer readCustomer(String id) { ... }
}
```
메서드 수준 인가가 활성화되면, `MyCustomerService#readCustomer` 호출은 다음과 같은 방식으로 진행될 수 있다: 

![메서드 수준 인가 작동 방식](/assets/images/posts_img/spring-security/request-and-method-authorization/methodsecurity.png)

1. Spring AOP는 `readCustomer`의 프록시 메서드를 호출한다. 프록시의 다른 어드바이저들 중에서 [`@PreAuthorize` 포인트컷](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html#annotation-method-pointcuts)과 일치하는 [`AuthorizationManagerBeforeMethodInterceptor`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/authorization/method/AuthorizationManagerBeforeMethodInterceptor.html)를 호출한다.
2. 인터셉터는 [`PreAuthorizeAuthorizationManager#check`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/authorization/method/PreAuthorizeAuthorizationManager.html)를 호출한다.
3. 인가 매니저(`AuthorizationManager`)는 `MethodSecurityExpressionHandler`를 사용해 어노테이션의 SpEL 표현식을 분석하고, `Supplier<Authentication>`과 `MethodInvocation`을 포함하는 `MethodSecurityExpressionRoot`로부터 이에 상응하는 `EvaluationContext`를 구성한다.
4. 인터셉터(`AuthorizationManagerBeforeMethodInterceptor`)는 이 컨텍스트(`EvaluationContext`)를 사용해 표현식을 평가한다; 구체적으로, `Supplier`에서 `Authentication`을 읽어와서 이 `Authentication`이 가진 권한 컬렉션(보통 `GrantedAuthority` 객체들의 컬렉션)에 `permission:read`가 포함되어 있는지 확인한다.
5. 보안 검사(권한 확인)가 통과하면, Spring AOP는 메서드 호출을 진행한다.
6. 통과하지 못하면, 인터셉터는 `AuthorizationDeniedEvent`를 발행하고 `AccessDeniedException`을 던진다. 이 예외는 [`ExceptionTranslationFilter`](https://ijnooyah.github.io/spring-security/handling-exceptions/#%EF%B8%8F-%EC%98%88%EC%99%B8-%EC%B2%98%EB%A6%AC)가 잡아서 응답으로 403 상태 코드를 반환한다.
7. 메서드 실행이 완료된 후, Spring AOP는 `@PostAuthorize` 포인트컷과 일치하는 [`AuthorizationManagerAfterMethodInterceptor`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/authorization/method/AuthorizationManagerAfterMethodInterceptor.html)를 호출한다. 이는 앞에와 동일한 방식으로 작동하지만 [`PostAuthorizeAuthorizationManager`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/authorization/method/PostAuthorizeAuthorizationManager.html)를 사용한다.
8. 평가가 통과되면(이 경우, 반환된 값이 로그인한 사용자에게 속한 것이라면) 처리가 정상적으로 계속된다.
9. 통과되지 않으면, 인터셉터는 `AuthorizationDeniedEvent`를 발행하고 `AccessDeniedException`을 던진다. 이 예외는 `ExceptionTranslationFilter`가 잡아서 응답으로 403 상태 코드를 반환한다. 

---

## 🔬 @PreAuthorize를 사용한 메서드 호출 인가
`@PreAuthorize` 어노테이션은 메서드가 실행되기 전에 특정한 보안 조건이 충족되는지 확인하는 데 사용된다. 보통 서비스 또는 컨트롤러 레이어의 메소드에 적용되어 해당 메서드가 호출되기 전에 사용자의 인증 정보와 권한을 검사한다.
```java
@PreAuthorize("hasRole('ADMIN')")
public Account readAccount(Long id) {
  // 'ROLE_ADMIN' 권한을 가진 인증된 사용자만 메서드를 호출할 수 있다.
}
```
`hasRole('ADMIN')` 표현식이 통과될 때만 메서드가 호출될 수 있다.

---

## 🏷️ @PostAuthorize를 사용한 메서드 결과 인가
`@PostAuthorize` 어노테이션은 메서드가 실행된 후에 보안 검사를 수행하는 데 사용된다.  

`@PreAuthorize`와는 달리, `@PostAuthorize`는 메서드 실행 후 결과에 대한 보안 조건을 검사하여 특정 조건을 만족하는 경우에만 사용자가 결과를 받을 수 있도록 한다.

```java
@PostAuthorize("returnObject.owner == authentication.name")
public Account readAccount(Long id) {
  // 계정이 로그인된 사용자에게 속하는 경우에만 반환된다.
}
```
`returnObject.owner == authentication.name` 표현식이 통과될 때만 메서드가 값을 반환할 수 있다. `returnObject`는 반환될 `Account` 객체다.

---

## 🧲 @PreFilter를 사용한 메서드 매개변수 필터링
`@PreFilter` 어노테이션은 메서드가 실행되기 전에 전달된 컬렉션 타입의 파라미터에 대한 필터링하는데 사용된다. 주로 사용자가 보내온 컬렉션(배열, 리스트, 맵, 스트림) 내의 객체들을 특정 기준에 따라 필터링하고 그 중 보안 조건을 만족하는 객체들에 대해서만 메소드가 처리하도록 할 때 사용된다.

```java
@PreFilter("filterObject.owner == authentication.name")
public Collection<Account> updateAccounts(Account... accounts) {
  // 'accounts'에는 로그인된 사용자가 소유한 계정만 포함된다.
  return updated;
}
```
`filterObject.owner == authentication.name` 표현식이 실패한 값을 `accounts`에서 제거하도록 필터링한다. `filterObject`는 `accounts`의 각 계정을 나타낸다.


위의 `updateAccounts` 메서드는 아래 네 개와 동일하게 작동한다.
```java
@PreFilter("filterObject.owner == authentication.name")
public Collection<Account> updateAccounts(Account[] accounts)

@PreFilter("filterObject.owner == authentication.name")
public Collection<Account> updateAccounts(Collection<Account> accounts)

@PreFilter("filterObject.value.owner == authentication.name")
public Collection<Account> updateAccounts(Map<String, Account> accounts)

@PreFilter("filterObject.owner == authentication.name")
public Collection<Account> updateAccounts(Stream<Account> accounts)
```
---

## 🧹 @PostFilter를 메서드 결과 필터링
`@PostFilter` 어노테이션은 메서드가 반환하는 컬렉션 타입의 결과에 대해 필터링을 수행하는 데 사용된다. 메서드가 컬렉션을 반환할 때 반환되는 각 객체가 특정 보안 조건을 충족하는지 확인하고 조건을 만족하지 않은 객체들을 결과에서 제거한다.

```java
@PostFilter("filterObject.owner == authentication.name")
public Collection<Account> readAccounts(String... ids) {
  // 리턴값은 로그인된 사용자가 소유한 계정만 포함된다.
  return accounts;
}
```
리턴값에서 `filterObject.owner == authentication.name` 표현식이 실패한 값을 제거하도록 필터링한다. `filterObject`는 `accounts`의 각 계정을 나타낸다.

<!--
---
## SpEL(Spring Expression Language)
요청 수준 인가에서는 SpEL을 사용하는 것보다 구체적인 `AuthorizationManger`를 사용하는 것을 권장하지만, 메서드 수준 인가에서는 핵심적인 역할을 한다. -->

---

# 🤼 요청 수준 인가 VS 메서드 수준 인가

|  | 요청 수준 | 메서드 수준 |
| --- | --- | --- |
| **인가 유형** | 범위가 넓음(coarse-grained) | 세밀함 (fine-grained) |
| **설정 위치** | 설정 클래스에서 선언됨 | 메서드 선언부에 정의됨 |
| **설정 스타일** | DSL(Domain-Specific Language) | Annotations |
| **인가 정의** | programmatic | SpEL |

둘의 사용은 인가 규칙을 어디에 두고 싶은지에 달려 있다.

> 메서드 수준 인가를 사용하는 경우, 어노테이션이 없는 메서드는 보호되지 않는다는 점을 주의해야 한다. 이를 방지하기 위해 `HttpSecurity` 인스턴스에서 포괄적인 인가 규칙(요청 수준 인가)을 선언하는 것이 좋다.

---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}

