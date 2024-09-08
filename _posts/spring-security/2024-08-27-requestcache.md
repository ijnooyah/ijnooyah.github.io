---
title: "[Spring Security] 인증 간 요청 저장"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security, 스프링 시큐리티 시리즈]

permalink: /spring-security/requestcache/

toc: true
toc_sticky: true

date: 2024-08-27
last_modified_at: 2024-08-29
---

{% include spring-security-list.md %}

---
# 🗂️ RequestCache
인증이 필요한 서비스에 접근하려 할 때, 사용자가 아직 로그인하지 않은 상태라면 로그인 페이지로 리다이렉트된다. 로그인이 성공하면 원래 요청했던 페이지로 돌아가야 한다.  
이처럼, 인증이 필요한 자원에 대한 요청을 일시적으로 저장했다가, 인증후 다시 요청해야하는 경우가 발생한다.  

스프링 시큐리티는 이러한 상황을 해결하기 위해 `RequestCache` 인터페이스를 제공한다.  
`RequestCache`는 `HttpServletRequest` 객체를 저장하고 필요에 따라 다시 꺼내는 역할을 한다.

기본적인 구현체로 `HttpSessionRequestCache`가 사용된다.

## 🌊 RequestCache 인증 흐름도

<div class="mermaid">
graph TD
    A[시작] --> B{인증 필요?}
    B -->|예| C[ExceptionTranslationFilter]
    C --> D[RequestCache에 HttpServletRequest 저장]
    D --> E[로그인 페이지로 리다이렉트]
    E --> F[사용자 로그인]
    F --> G{인증 성공?}
    G -->|예| H[RequestCacheAwareFilter]
    H --> I[RequestCache에서 저장된 요청 검색]
    I --> J[원래 요청 재실행]
    J --> K[종료]
    
    B -->|아니오| K
    G -->|아니오| E

    subgraph RequestCache 처리
    D
    I
    end
</div>

**1.** 시작점에서 인증이 필요한지 확인한다.

**2.** 인증이 필요한 경우:
1. [`ExceptionTranslationFilter`](https://ijnooyah.github.io/spring-security/handling-exceptions/#%EF%B8%8F-%EC%98%88%EC%99%B8-%EC%B2%98%EB%A6%AC){: target="_blank"}가 작동한다.
2. `HttpServletRequest`를 `RequestCache`에 저장한다.
3. 사용자를 로그인 페이지로 리다이렉트한다.

**3.** 사용자가 로그인을 시도한다.

**4.** 인증이 성공하면:
1. [`RequestCacheAwareFilter`](https://docs.spring.io/spring-security/site/docs/6.3.3/api/org/springframework/security/web/savedrequest/RequestCacheAwareFilter.html){: target="_blank"}가 작동한다
2. `RequestCache`에 저장된 원래 요청을 가져온다.
3. 원래 요청을 재실행한다.

**5.** 인증이 실패하면 다시 로그인 페이지로 돌아간다.

**6.** 인증이 필요없거나 프로세스가 완료되면 종료된다.


---

<p class="ref">참고</p>
- [스프링 시큐리티 공식문서](https://docs.spring.io/spring-security/reference/servlet/architecture.html#requestcache){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}

