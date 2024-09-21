---
title: "[Spring Security] OAuth2 실습 - Swagger 구현하기"
excerpt: ""

categories:
  - Spring Security
tags:
  - [Spring Security, OAuth2, 실습, OAuth2 시리즈, Swagger]

permalink: /spring-security/implementing-swagger

toc: true
toc_sticky: true

date: 2024-09-21
last_modified_at: 2024-09-21
---



{% include oauth2-list.md %}

---

공통응답 CommonResponse도 적용해뒀으므로 [깃허브 전체 코드](https://github.com/ijnooyah/oauth2-spring-security)를 참고하길 바란다.

커스텀 필터 API를 스웨거로 적용하는 것은 이 [글](https://somuchthings.tistory.com/298)을 참고해서 구현했다.

# 🎨 커스텀 필터 API Swagger

```java
@Configuration
public class SwaggerConfig {

    private static final String LOGIN_WITH_EMAIL = "/api/v1/auth/login";
    private final ApplicationContext applicationContext;

    public SwaggerConfig(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    public OpenAPI openAPI() {
        Info info = new Info()
                .version("1.0")
                .title("OAuth2 & Spring Security 6")
                .description("스프링 시큐리티 버전 6으로 OAuth2 구현하기");

        return new OpenAPI()
                .info(info);
    }

    @Bean
    public OpenApiCustomizer springSecurityLoginEndpointCustomiser() {
        FilterChainProxy filterChainProxy = applicationContext.getBean(
                AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME, FilterChainProxy.class);
        return openAPI -> {
            for (SecurityFilterChain filterChain : filterChainProxy.getFilterChains()) {
                Optional<RestAuthenticationFilter> optionalFilter =
                        filterChain.getFilters().stream()
                                .filter(RestAuthenticationFilter.class::isInstance)
                                .map(RestAuthenticationFilter.class::cast)
                                .findAny();
                if (optionalFilter.isPresent()) {
                    RestAuthenticationFilter restAuthenticationFilter = optionalFilter.get();
                    Operation operation = new Operation();
                    Schema<?> schema = new ObjectSchema()
                            .addProperty(restAuthenticationFilter.getEmailParameter(), new StringSchema())
                            .addProperty(restAuthenticationFilter.getPasswordParameter(),
                                    new StringSchema());
                    RequestBody requestBody = new RequestBody().content(
                            new Content().addMediaType(org.springframework.http.MediaType.APPLICATION_JSON_VALUE,
                                    new MediaType().schema(schema)));
                    operation.requestBody(requestBody);
                    ApiResponses apiResponses = new ApiResponses();
                    apiResponses.addApiResponse(String.valueOf(HttpStatus.OK.value()),
                            new ApiResponse().description(HttpStatus.OK.getReasonPhrase()));
                    apiResponses.addApiResponse(String.valueOf(HttpStatus.BAD_REQUEST.value()),
                            new ApiResponse().description(HttpStatus.BAD_REQUEST.getReasonPhrase()));
                    operation.responses(apiResponses);
                    operation.addTagsItem("email-login-endpoint");
                    PathItem pathItem = new PathItem().post(operation);
                    openAPI.getPaths().addPathItem(LOGIN_WITH_EMAIL, pathItem);
                }
            }
        };
    }
}
```

솔직히 버전에 맞게 몇개만 바꿔주고 참고한 블로그에 있는 코드를 거의 가져다 썼다.
또한 이 코드들을 전부 분석하지 못했다. 나중에 우선순위들을 다 끝내면 분석해보는 시간을 가져보는게 좋을 것 같다.

OAuth2 로그인 엔드포인트를 구글에 찾아보고 AI에게도 물어봤는데 일단 내가 찾아본 바로는 이거다 싶은 정보가 없는 것 같다.
테스트해보려면 프론트엔드에 구현하는 방법이 최선인듯하다.

---

<p class="ref">참고</p>
- [https://somuchthings.tistory.com/298](https://somuchthings.tistory.com/298){: target='_blank'}