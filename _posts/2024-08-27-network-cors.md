---
title: "[Network] CORS"
excerpt: ""

categories:
  - Network
tags:
  - [Network]

permalink: /network/cors/

toc: true
toc_sticky: true

date: 2024-08-27
last_modified_at: 2024-08-27
---
# 🔄 CORS란?
CORS(Cross-Origin Resource Sharing)는 웹 브라우저에서 보안을 위해 실행되는 정책이다.   
한국어로 직역하면 '교차 출처 리소스 공유'라고 해석할 수 있다. 여기서 '교차 출처(Cross-Origin)'란 '다른 출처'를 의미한다.  
해석을 해보면 Cross-Origin의 Resource를 공유하는 정책이라고 볼 수 있다.  

이 정책은 한 출처(origin)에서 실행 중인 웹 애플리케이션이 다른 출처의 리소스에 접근할 때 적용된다.

## 🌍 출처(Origin)
출처는 프로토콜(http, https), 도메인(example.com, localhost), 포트번호(80, 443)로 구성된다. 
- `https://example.com`  
- `http://localhost:3000`

이 두 URL은 서로 다른 출처(Origin)다.

## 🔍 CORS의 등장배경
브라우저는 기본적으로 SOP 정책을 따르고 있다.
<blockquote class="info">
<strong>SOP(Same-Origin Policy)</strong>
<br>
SOP는 2011년 RFC 6454에서 등장한 보안 정책으로, 동일한 출처(프로토콜, 도메인, 포트)에서 온 리소스만 접근할 수 있도록 제한하는 하는 보안 정책이다.  
즉, <strong>"같은 출처에서만 리소스를 공유할 수 있다"</strong>라는 규칙을 가진 정책이라 할 수 있다.
</blockquote>

SOP는 보안을 위해 같은 출처의 리소스만 접근을 허용한다.  
하지만 현대 웹 개발에서는 다른 출처의 리소스를 사용해야 하는 경우가 많아졌다. 이러한 필요성과 보안 사이의 균형을 맞추기 위해 CORS가 도입되었다.

그러나 보안 도 중요하지만 개발을 하다 보면 기능상 어쩔 수 없이 다른 출처 간의 상호작용을 해야 하는 케이스가 존재한다.  
'다른 출처'의 API 서버를 두거나, '다른 출처'의 외부 리소스를 가져다 쓰는 경우가 있기 때문이다.

"CORS를 지키면, '다른 출처'에서도 데이터를 불러올 수 있게 해줄게!"

[SOP, CORS는 보안에서 왜 필요할까?](https://velog.io/@hunjison/SOP-CORS%EB%8A%94-%EB%B3%B4%EC%95%88%EC%97%90%EC%84%9C-%EC%99%9C-%ED%95%84%EC%9A%94%ED%95%A0%EA%B9%8C){: target="_blank"}

## 📜 CORS 정책은 어제 검사할까?
![CORS 정책은 어제 검사할까?](/assets/images/posts_img/network/cors/when.png)

여기서 중요한 사실 한 가지는 이렇게 **출처를 비교하는 로직이 서버에 구현된 스펙이 아니라 <mark>브라우저</mark>에 구현되어 있는 스펙**이라는 것이다. 서버 간 통신에서는 CORS가 적용되지 않는다.

만약 우리가 CORS 정책을 위반하는 리소스 요청을 하더라도 해당 서버가 같은 출처에서 보낸 요창만 받겠다는 로직을 가지고 있는 경우가 아니라면
서버는 정상적으로 응답을 하고, 이후 브라우저가 이 응답을 분석해서 CORS 정책 위반이라고 판단되면 그 응답을 사용하지 않고 그냥 버리는 순서인 것이다.

서버는 CORS를 위반하더라도 정상적으로 응답을 해주고, 응답의 파기 여부는 브라우저가 결정한다.
즉, CORS는 브라우저의 구현 스펙에 포함되는 정책이기 때문에, 브라우저를 통하지 않고 서버 간 통신을 할 때는 이 정책이 적용되지 않는다.  
또한 CORS 정책을 위반하는 리소스 요청 때문에 에러가 발생했다고 해도 서버 쪽 로그에는 정상적으로 응답을 했다는 로그만 남기 때문에, CORS가 돌아가는 방식을 정확히 모르면 에러 트레이싱에 난항을 겪을 수 도 있다.

---

# 📈 CORS의 동작 방식
CORS는 세가지 시나리오로 동작한다.
- [Simple Request](#-simple-request)
- [Preflight Request](#️-preflight-request)
- [Credential Request](#-credential-request)

## ✅ Simple Request
![Simple Request](/assets/images/posts_img/network/cors/simple-request.png)

단순 요청은 다음 조건을 모두 만족해야 한다.
- HTTP 메서드가 `GET`, `HEAD`, `POST` 중 하나일 것
- 자동으로 설정되는 헤더 외에, 수동으로 설정할 수 있는 헤더는 `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`만 가능하다. (이들 외의 커스텀 헤더 사용이 제한됨)
- `Contet-Type` 헤더는 다음 값들만 허용된다.
   - `application/x-www-form-urlencoded`
   - `multipart/form-data`
   - `text/plain`

이 조건을 만족하면, 브라우저는 바로 요청을 보내고 서버의 응답을 확인한다.

하지만 Simple Request는 실제 웹 개발에서는 거의 충족시키기 어렵다.  
1. **Content-Type 제한**: 대부분의 현대 API는 `application/json`을 사용하지만, 이는 Simple Request에 허용되지 않는다.  
2. **추가 헤더 사용**: 많은 웹 애플리케이션이 `Authorization` 등 추가 헤더를 사용하며, 이는 Simple Request 조건을 벗어난다.  
3. **HTTP 메소드 제한**: `GET` 외의 메소드나 특정 MIME Type을 사용하는 `POST` 요청은 서버에 부작용을 일으킬 수 있어, 브라우저가 Preflight Request를 강제한다.

## ✈️ Preflight Request
Simple Request 조건을 만족하지 않는 경우, 브라우저는 예비 요청을 먼저 보낸다.  
이 예비 요청은 HTTP `OPTIONS` 메서드를 사용하며, 실제 요청이 안전한지 확인한다.

예를 들어, POST 요청을 application/json 컨텐츠 타입으로 보내려고 할 때 예비 요청이 발생한다.
```http
OPTIONS /api/data HTTP/1.1
Origin: https://example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type
```
서버는 이에 대해 허용되는 메서드와 헤더를 응답한다:
```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
```
위와 같은 응답을 받은 후에야 브라우저는 실제 요청을 보낸다.
- `Access-Control-Allow-Origin` : 허가된 Origin  
- `Access-Control-Allow-Methods` : 허가된 메서드  
- `Access-Control-Allow-Headers` : 허가된 헤더  
- `Access-Control-Max-Age` : 응답 캐시가 유효 시간  

<br>
preflight 시나리오의 플로우를 자세히 살펴보면 아래와 같다.

![Preflight Request](/assets/images/posts_img/network/cors/preflight2.png)

1. 브라우저는 위 XMLHttpRequest가 Cross-Origin 요청인 것을 판단하여 아래와 같이 요청에 "Origin: https://foo.example" 헤더를 추가한다. 또한 브라우저는 `POST` 방식이며, `Content-Type`이 `application/x-www-form-urlencoded`, `multipart/form-data`, `text/plain`에 포함되지 않기 때문에 Prefight Request 방식으로 보내야 한다는 것을 알고 있다. 그래서 브라우저는 요청에 아래와 같이 헤더 정보를 추가하여 외부 서버로 Preflight Request(예비 요청)을 보낸다.

2. 서버는 이 Preflight Request(예비 요청)에 대한 응답으로 현재 자신이 어떤 것들을 허용하고 있는지에 대한 정보를 response header에 담아서 브라우저에게 다시 보내주게 된다.

3. 응답으로 받은 response header의 정보를 통해서 브라우저는 본 요청을 외부 서버로 보낼지 말지를 판단하게 된다. 위 예시에서 표시된 정보는 '해당 API가 Cross-Origin에 대해서 `POST`, `GET`, `OPTIONS`와 커스텀 헤더인 `X-PINGOTHER` 그리고 `Content-Type`을 허용한다'는 의미이다. 이에 해당되는 것들은 안전하다고 판단하여 CORS 위반으로 간주하지 않고, 브라우저는 `POST`인 본 요청을 브라우저는 외부 서버로 보낸다.

> <b>🚨 주의할 점!</b>  
예비 요청이 성공적으로 완료되어 `200` 상태 코드를 반환하더라도, 실제 요청 시 CORS 정책 위반 에러가 발생할 수 있다. 이는 CORS 정책 위반 여부가 예비 요청의 성공 여부와 직접적인 관련이 없기 때문이다.<br>  
브라우저는 예비 요청에 대한 응답을 받은 후에 CORS 정책 위반 여부를 판단한다. 이때 가장 중요한 판단 기준은 응답 헤더에 포함된 `Access-Control-Allow-Origin`값의 유효성이다. 만약 예비 요청 자체가 실패해 `200`이 아닌 상태 코드를 반환하더라도, 응답 헤더에 올바른 `Access-Control-Allow-Origin`값이 포함되어 있다면 CORS 정책 위반으로 간주되지 않는다.

> 참고로 Preflight Request 방식은 많은 리소스를 잡아 먹는다.  
그렇기 때문에 서버에서 "Access-Control-Max-Age" 헤더 정보를 통해서 Preflight Request 를 캐싱함으로써 그 효율을 높힐 수 있다.  
[[Web] CORS 동작 방식과 해결 방법](https://ingg.dev/cors/#cors_solution){: target='_blank'}
{: .info}

> 응답 헤더와 관련해서는 다음 정보들을 참고해보자.  
[(HTTP) 알아둬야 할 HTTP 응답 헤더](https://www.zerocho.com/category/HTTP/post/5b4c4e3efc5052001b4f519b){: target='_blank'}  
[[프론트엔드] CORS의 유래 및 특징, 활용](https://velog.io/@kansun12/%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-CORS){: target='_blank'}  
[CORS](https://koguri.tistory.com/65){: target='_blank'}
{: .info}
  
## 🔑 Credential Request
이 시나리오는 헤더에 인증과 관련된 정보(쿠키, 토큰 등)를 담아서 보내는 Credential Request (인증된 요청)을 사용하는 방법이다.  
CORS의 기본적인 방식이라기 보다는 다른 출처 간 통신에서 좀 더 보안을 강화하고 싶을 때 사용한다.

예를 들어, 자바스크립트의 fetch API를 사용하거나 Axios, Ajax 등을 사용할 때 서버로 쿠키를 함께 전송해야 하는 경우가 있는데, 이런 경우 요청에 인증 정보가 포함되므로 Credential Request 허용이 필요하다.

클라이언트 측에서 요청을 보낼 때 'credentials'옵션을 설정해 인증 정보 포함 여부를 제어할 수 잇다.
이 옵션에는 총 3가지의 값을 사용할 수 있으며, 각 값들이 가지는 의미는 다음과 같다.

| 옵션 값	 | 설명 |
| --- | --- |
| same-origin (기본값) | 같은 출처 간 요청에만 인증 정보를 담을 수 있다 | 
| include |모든 요청에 인증 정보를 담을 수 있다 | 
| omit | 모든 요청에 인증 정보를 담지 않는다 | 

Credential Request를 허용하려면 서버 측에서도 특별한 설정이 필요하다. 먼저, 응답 헤더에 `Access-Control-Allow-Credentials: true`를 포함해야 한다. 또한 `'Access-Control-Allow-Origin`헤더에 와일드카드(*)를 사용할 수 없으며, https://foo.com과 같이 출처를 명시해야한다.

---

# ⚙️ CORS 에러 해결방법
<b>Spring Security 사용</b>  

CORS 설정은 스프링 시큐리티 이전에 처리되어야 한다. 
`CorsConfigurationSource`를 Bean으로 등록하여 설정할 수 있다.
```java
@Bean
CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
    configuration.setAllowedMethods(Arrays.asList("GET","POST","PUT","DELETE"));
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    return source;
}
```
이 방법은 스프링 시큐리티의 필터 체인에 CORS 설정을 통합하여, 보안 검사 이전에 CORS 처리가 이루어지도록 한다.

스프링 시큐리티의 CORS 지원에 대한 자세한 내용은 [공식문서](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html){: target='_blank'}를  참고하자.

><b>❓ 왜 CORS 설정은 스프링 시큐리티 이전에 처리되어야 할까</b>  
CORS 설정이 스프링 시큐리티 이전에 처리되어야 하는 주된 이유는 브라우저의 사전 요청(Preflight Request) 때문이다. 사전 요청에는 쿠키(JSESSIONID)가 포함되어 있지 않다. 요청에 쿠키가 없으면 스프링 시큐리티가 먼저 처리할 경우, 사용자가 인증되지 않은 것으로 간주해 요청을 거부할 수 있다.

이 외에 프록시 사용, 브라우저 확장 프로그램, 서버리스 함수 사용 등이 있다.

---
<p class="ref">참고</p>
- [https://velog.io/@effirin/CORS란-무엇인가](https://velog.io/@effirin/CORS%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80){: target="_blank"}
- [스프링 시큐리티 완전 정복](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%8B%9C%ED%81%90%EB%A6%AC%ED%8B%B0-%EC%99%84%EC%A0%84%EC%A0%95%EB%B3%B5/dashboard){: target="_blank"}
- [스프링 시큐리티 공식 문서](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html){: target='_blank'}


