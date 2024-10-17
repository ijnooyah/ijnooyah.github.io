---
title: "[Swagger] MultipartFile 업로드 스웨거로 문서화 하기"
excerpt: ""

categories:
  - Swagger
tags:
  - [Swagger, 트러블 슈팅]

permalink: /swagger/multipartfile-trouble-shooting

toc: true
toc_sticky: true

date: 2024-10-16
last_modified_at: 2024-10-16
---

실습 프로젝트를 하면서 프로필 이미지를 올리는 기능을 추가하였는데 스웨거로 문서화 하면서 겪었던 문제들을 해결해 나간 과정을 기록해 보려한다.

# 🚀 초기 구현 
## 🤔 문제 상황 
처음에는 단순히 DTO에 `MultipartFile` 타입의 `profileImage` 필드를 추가하는 방식으로 구현을 시작했다. 그러나 이방식으로 하게되면 스웨거 UI에서는 `profileImage` 필드가 JSON 형식의 텍스트 입력 필드로 표시된다.

이는 파일 업로드를 위한 적절한 인터페이스가 아니기 때문에 방법을 찾아보았다.

## 🎉 해결 방법 
컨트롤러 메서드에 `consumes` 속성을 추가했다. 
```java
@PostMapping(value = "/signup", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
// ...
```
`consumes = MediaType.MULTIPART_FORM_DATA_VALUE` 속성은 이 엔드포인트가 `multipart/form-data` 타입의 요청만을 처리하도록 명시한다. 이를 통해 스웨거는 파일 업로드 UI를 보여준다.

![alt text](/assets/images/posts_img/swagger/multipartfile-trouble-shooting/solution1.png)

파일 업로드 인터페이스를 원하는 대로 표시하는 데는 성공했지만, 실행 테스트를 하는 과정에서 또 새로운 문제가 발생했다.

---

# 📦 요청 처리 시 직렬화 문제 
## 🤔 문제 상황
스웨거에서 실제로 요청을 보내면 서버에서 에러가 발생했다.  
이는 `MultipartFile`을 포함한 DTO를 JSON으로 직렬화 하려는 시도 때문이었다. DTO를 JSON으로 직렬화하려고 할 때 `MultipartFile`은 JSON으로 직렬화를 할 수 없는 것이다.

## 🎉 해결 방법
`@RequestPart`를 통해  `MultipartFile`을 별도로 처리하도록 변경했다.

```java
@PostMapping(value = "/signup", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public CommonResponse<UserResponse> signup(
        @RequestPart("signUpRequest") SignUpRequest signUpRequest,
        @RequestPart(value = "profileImage", required = false) MultipartFile profileImage,
        HttpServletRequest request,
        HttpServletResponse response) throws IOException {

    // 로직
}
```

이렇게 변경하면
1. `SignUpRequest`는 JSON 형식으로 전송되어 자동으로 객체로 변환된다.
2. `profileImage`는 별도의 파일 파트로 처리된다.

---

# ⚖️ 스웨거와 포스트맨 간의 동작 차이
## 🤔 문제 상황
위의 문제까지 해결 후, 포스트맨에서는 API가 정상적으로 동작했지만 스웨거에서 테스트할 때는 여전히 문제가 발생했다.
```
Content-Type 'application/octet-stream' is not supported
```
위와 같은 에러메세지가 났는데, 이 문제의 원인은 스웨거와 포스트맨이 `multipart/form-data` 요청을 처리하는 방식의 차이에 있었다. 포스트맨에서는 각 파트의 Content-Type을 명시적으로 설정할 수 있지만, 스웨거에서는 그렇지 않은 것이다.  
`multipart/form-data` 요청에서 각 파트의 기본 Content-Type은 `application/octet-stream`이다.  
스웨거는 이 기본값을 사용해 요청을 보내지만 서버 측에선 이걸 처리할 준비가 되지 않은 것이다.

## 🎉 해결
`application/octet-stream` Content-Typea을 처리할 수 있는 커스텀 `HttpMessageConverter`를 구현해 해결했다.  

```java
@Component
public class MultipartJackson2HttpMessageConverter extends AbstractJackson2HttpMessageConverter {

    /**
     * "Content-Type: multipart/form-data" 헤더를 지원하는 HTTP 요청 변환기
     */
    public MultipartJackson2HttpMessageConverter(ObjectMapper objectMapper) {
        super(objectMapper, MediaType.APPLICATION_OCTET_STREAM);
    }

    @Override
    public boolean canWrite(Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    public boolean canWrite(Type type, Class<?> clazz, MediaType mediaType) {
        return false;
    }

    @Override
    protected boolean canWrite(MediaType mediaType) {
        return false;
    }
}
```
해당 컨버터는 `application/octet-stream` Content-Type의 요청 본문(body)을 JSON으로 파싱할 수 있게 해준다. 

> 참고: `canWrite` 메서드들을 오버라이드해 `false`를 반환하는 이유는 이 컨버터가 읽기 전용으로만 사용되어야 하기 때문이다. 

---
<p class='ref'>참고</p>
- [https://devsungwon.tistory.com/entry/Spring-MultipartFile%EC%9D%B4-%ED%8F%AC%ED%95%A8%EB%90%9C-DTO-requestBody%EB%A1%9C-%EC%9A%94%EC%B2%AD%EB%B0%9B%EA%B8%B0-swagger-%EC%9A%94%EC%B2%AD](https://devsungwon.tistory.com/entry/Spring-MultipartFile%EC%9D%B4-%ED%8F%AC%ED%95%A8%EB%90%9C-DTO-requestBody%EB%A1%9C-%EC%9A%94%EC%B2%AD%EB%B0%9B%EA%B8%B0-swagger-%EC%9A%94%EC%B2%AD){: target='_blank'}





