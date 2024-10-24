---
title: "[Spring Boot] 문자열로 들어오는 날짜를 LocalDateTime으로 받는 법"
excerpt: ""

categories:
  - Spring Boot
tags:
  - [Spring Boot, DateTimeFormat, JsonFormat, 트러블 슈팅]

permalink: /spring-boot/string-to-local-date-time

toc: true
toc_sticky: true

date: 2024-10-03
last_modified_at: 2024-10-03
---

# 🕰️ 날짜 타입으로 직렬화/역직렬화 

문자열로 들어오는 날짜를 어떻게 `LocalDate`나 `LocalDateTime` 등으로 받을 수 있을까?

다음과 같이 문자열로 들어오는 값을 `LocalDateTime` 타입 필드(`startDate`, `endDate`)가 받으면 에러가 난다.
```
2024-10-03T15:11:49.686+09:00  WARN 14632 --- [nio-8080-exec-9] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public com.yoonji.adminproject.common.dto.response.CommonResponse<com.yoonji.adminproject.admin.dto.response.AdminUserListResponse> com.yoonji.adminproject.admin.controller.AdminUserController.searchUsersWithCursor(com.yoonji.adminproject.admin.dto.request.AdminUserSearchCondition) with 2 errors: [Field error in object 'adminUserSearchCondition' on field 'endDate': rejected value [2024-10-01 00:00:00]; codes [typeMismatch.adminUserSearchCondition.endDate,typeMismatch.endDate,typeMismatch.java.time.LocalDateTime,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [adminUserSearchCondition.endDate,endDate]; arguments []; default message [endDate]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.time.LocalDateTime' for property 'endDate'; Failed to convert from type [java.lang.String] to type [@io.swagger.v3.oas.annotations.media.Schema java.time.LocalDateTime] for value [2024-10-01 00:00:00]]] [Field error in object 'adminUserSearchCondition' on field 'startDate': rejected value [2024-09-01 00:00:00]; codes [typeMismatch.adminUserSearchCondition.startDate,typeMismatch.startDate,typeMismatch.java.time.LocalDateTime,typeMismatch]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [adminUserSearchCondition.startDate,startDate]; arguments []; default message [startDate]]; default message [Failed to convert property value of type 'java.lang.String' to required type 'java.time.LocalDateTime' for property 'startDate'; Failed to convert from type [java.lang.String] to type [@io.swagger.v3.oas.annotations.media.Schema java.time.LocalDateTime] for value [2024-09-01 00:00:00]]] ]
```

그렇다면 `startDate`, `endDate`의 타입을 `String`으로 해서 `LocalDateTime`으로 변환해야 하는 것인가?

아니다. 처음부터 `LocalDateTime`으로 받을 수 있는 방법이 있다. 그걸 알아보도록 하자.

---
{: .style1}

## 🔗 URL 파라미터 처리

Spring에서 URL 파라미터를 받는 방법은 크게 2가지가 있다.
- `@ModelAttribute`를 사용해서 DTO 객체로 받기
- `@RequestParam`를 사용해서 필드별로 받기

<b>@ModelAttribute</b>  

```java
@GetMapping("/get")
public String get(LocalDateDto dto) {
    return "mission complete";
}

@Getter
@RequiredArgsConstructor
public class LocalDateDto {
    private final String name;
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private final LocalDateTime dateTime;
}
```

> 별도의 어노테이션을 지정하지 않으면 기본적으로 스프링은 `@ModelAttribute`를 할당한다. 

<b>@RequestParam</b> 

```java
@GetMapping("/requestParameter")
public String get(
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @RequestParam(name = "dateTime") LocalDateTime dateTime) {
    return "mission complete";
}
```

위와 같이 URL 파라미터로 날짜를 받을 때는 `@DateTimeFormat` 어노테이션을 사용한다. 

---
{: .style1}

## 📩 Request Body 처리

```java
@PostMapping("/post")
public String post(@RequestBody LocalDateJsonDto localDateJsonDto) {
    return "post mission complete";
}

@Getter
@NoArgsConstructor
public class LocalDateJsonDto {
    private String name;
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime dateTime;
}
```

POST 요청의 Request Body에서 날짜를 처리할 때는 `@JsonFormat` 어노테이션을 사용한다. 이는 Jackson 라이브러리가 처리한다.

---
{: .style1}

## 📤 Response Body 처리

```java
@GetMapping("/response")
public LocalDateJsonDto response() {
    return new LocalDateJsonDto("user", LocalDateTime.now());
}

@Getter
@NoArgsConstructor
@AllArgsConstructor
public class LocalDateJsonDto {
    private String name;
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
    private LocalDateTime dateTime;
}
```

Response Body에서 날짜를 원하는 형식으로 반환하려면 `@JsonFormat` 어노테이션을 사용한다.

---

# ⚖️ @DateTimeFormat vs @JsonFormat

| 특성 | @DateTimeFormat | @JsonFormat |
|-------|------------------|---------------|
| 제공 | Spring Framework | Jackson 라이브러리 |
| 용도 | URL 파라미터, 폼 데이터 등 비 JSON 데이터의 날짜 포맷 지정 | JSON 데이터의 직렬화/역직렬화 시 날짜 포맷 지정 |
| 사용 위치 | `@ModelAttribute`와 `@RequestParam`에서 사용 | Request Body와 Response Body에서 사용 |

- GET 요청 (URL 파라미터): `@DateTimeFormat` 사용
- Reqeust Body: `@JsonFormat` 사용
- Response Body: `@JsonFormat` 사용

---

<p class='ref'>참고</p>
- [https://github.com/sungwoon129/SpringBoot-LocalDateTime](https://github.com/sungwoon129/SpringBoot-LocalDateTime){: target='_blank'}

