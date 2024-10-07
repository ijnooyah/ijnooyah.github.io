---
title: "[Spring Framework] @GetMapping과 @RequestBody(feat. SpringDoc)"
excerpt: ""

categories:
  - Spring Framework
tags:
  - [Spring Framework]

permalink: /spring-framework/get-mapping

toc: true
toc_sticky: true

date: 2024-10-03
last_modified_at: 2024-10-03
---

# 🚨 문제 상황
실습 프로젝트를 진행하면서 겪은 `@GetMapping` 관련 오류와 해결 과정을 정리해보겠다.

다음과 같은 컨트롤러 메서드를 작성했다:
```java
@GetMapping("/search")
public CommonResponse<AdminUserListResponse> searchUsersWithCursor(@RequestBody AdminUserSearchCondition condition) {
    return new CommonResponse<>(HttpStatus.OK, adminUserService.searchUsersWithCursor(condition));
}
```

`/search` API가 많은 파라미터를 받아야 해서, `@RequestParameter`로 각각 바인딩하는 대신
`AdminUserSearchCondition`이라는 DTO를 만들어 사용했다. 그러나 스웨거를 통해 테스트를 해보니 파라미터 바인딩이 되지 않았다. 

```
2024-10-03T21:33:26.156+09:00  WARN 16752 --- [nio-8080-exec-9] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotReadableException: Required request body is missing: public com.yoonji.adminproject.common.dto.response.CommonResponse<com.yoonji.adminproject.admin.dto.response.AdminUserListResponse> com.yoonji.adminproject.admin.controller.AdminUserController.searchUsersWithCursor(com.yoonji.adminproject.admin.dto.request.AdminUserSearchCondition)]
```

`HttpMessageNotReadableException`이 발생했다.  

그 당시 `AdminUserSearchCondition`의 코드는 이랬다.  
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class AdminUserSearchCondition {
    private Long cursorId;
    private int size;
    private Sort.Direction sortDirection = Sort.Direction.ASC;

    private String searchInput;
    private String searchType;
    private Set<String> roles;
    private LocalDateTime startDate; // 가입일 범위 시작
    private LocalDateTime endDate; // 가입일 범위 끝
    private Set<String> providers; // 가입 경로

}
```

---

# 🕵️ 원인 분석 과정
<b>🔍 시도</b>  
처음에는 `startDate`와 `endDate`가 `LocalDateTime`인데 들어오는 값은 문자열이라 못받는건가 해서 (이것도 어찌보면 맞는 거긴한데 근본적인 문제는 아니였다. 이것과 관련된건 또 거기서 설명하겠다.) 일단 혹시 몰라서 `startDate`와 `endDate` 필드를 주석처리하고 실행해봤다. 하지만 그래도 똑같이 바인딩을 못하는 것이었다!

<b>💡 해결: @RequestBody 제거</b>  
`@RequestBody` 어노테이션의 문제라는 생각이 들어서 서치를 해보니 `@GetMapping`에는 `@RequestBody`를 쓰는게 아닌 것이다. 

> **❓ 여기서 알아보자 @RequestBody란 뭘까?**  
Spring Framework [공식 문서](https://docs.spring.io/spring-framework/reference/web/webflux/controller/ann-methods/requestbody.html)에서는 `@RequestBody`는 HTTP 요청 본문을 Java 객체로 변환하는 데 사용된다고 한다.    
HTTP 요청 본문 즉 body를 Java 객체로 변환하는 역할을 한다는 것이다.  
그런데?! `@GetMapping`에서 그러니깐 HTTP GET 메서드에서는 일반적으로 요청 본문(body)에 데이터를 포함하지 않는다.   
클라이언트에서 data 형식으로 보내더라도 쿼리파라미터 형식으로 변경되어서 전송이 된다.  

따라서 `@RequestBody` 어노테이션을 사용한 경우 값을 찾지 못한 것이다.  

결론적으로 `@ReqeustBody` 어노테이션을 제거했더니 해당 에러는 사라졌고 바인딩이 잘되었다. (`LocalDateTime` 타입인 `startDate`, `endDate`가 없다는 가정하에)

---

# ✅ 최종 코드

```java
@GetMapping("/search")
public CommonResponse<AdminUserListResponse> searchUsersWithCursor(@ParameterObject AdminUserSearchCondition condition) {
    return new CommonResponse<>(HttpStatus.OK, adminUserService.searchUsersWithCursor(condition));
}
```

완성된 코드에서 `@ParameterObject`는 뭘까? 이건 스웨거 관련된 코드인데 
`@RequestParameter`를 사용해서 파라미터를 하나하나 받게되면 스웨거 문서에 필드 하나하나 받을 수 있게 나온다.  
그런데 `AdminUserSearchCondition`만 달랑 놓으니 JSON 형태로 입력받을 수 있게 나오는데 이게 되게 보기에도 안좋았고 입력하기도 불편했다. 다른 사람의 이미지를 가져와서 어떻게 보여지는 보여주면  

![alt text](/assets/images/posts_img/spring-framework/get-mapping/json.png)
출처: https://yeonyeon.tistory.com/324

이런식으로 나오는 것이다. 

뭔가 방법이 없을까 하다가 `@ParameterObject`라는 것을 찾았고 해당 어노테이션을 붙이면 필드를 마치 쿼리 파라미터를 사용한 것처럼 하나하나 받을 수 있게 스웨거 문서를 구성할 수 있다.  

![alt text](/assets/images/posts_img/spring-framework/get-mapping/parameterobject.png)

이런식으로 말이다!