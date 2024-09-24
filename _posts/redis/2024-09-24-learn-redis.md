---
title: "[Redis] Spring Boot 프로젝트에 Redis 적용하기"
excerpt: "설명"

categories:
  - Redis
tags:
  - [Redis]

permalink: /redis/learn-redis

toc: true
toc_sticky: true

date: 2024-09-24
last_modified_at: 2024-09-24
---

# 🧱 프로젝트 소개
해당 프로젝트는 [OAuth2 실전 프로젝트](https://ijnooyah.github.io/spring-security/implementing-swagger)에 Redis 캐싱을 적용한 확장 버전이다.
Redis를 사용해 성능을 향상시키고 서버 부하를 줄이는 방법을 구현해 보도록 하자.

전체 코드는 [깃허브](https://github.com/ijnooyah/redis-spring-boot)에서 확인 가능하다.

---

# 📦 의존성 추가

먼저, `build.gradle` 파일에 Redis 관련 의존성을 추가해야 한다.
```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```
이 의존성은 Spring Boot에서 Redis를 쉽게 사용할 수 있게 해주는 스타터 패키지이다. 자동 설정과 필요한 클래스들을 제공한다.

---

# ⚙️ Redis 설정
Redis를 사용하기 위한 설정 클래스를 만들어 준다. 이 설정은 캐시 매니저를 생성하고 캐시의 기본 동작을 정의한다.

`RedisConfig`파일에 `cacheManger` 빈을 등록해준다.
```java
@Configuration
public class RedisConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30L))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        return RedisCacheManager
                .builder(connectionFactory)
                .cacheDefaults(cacheConfig)
                .transactionAware()
                .build();
    }

}
```
- `entryTtl(Duration.ofMinutes(3L))`: TTL을 30분으로 설정한다. 
- `serializeKeysWith`: 캐시 키를 문자열로 직렬화한다.
- `serializeValuesWith`: 값을 JSON 형식으로 직렬화한다.
- `transactionAware()`: 이 옵션은 트랜잭션이 성공적으로 완료된 후에만 캐시를 업데이트하도록 보장할 수 있다.

---
  
# 🔓 캐싱 활성화

애플리케이션에서 캐싱을 사용하려면 main 클래스에 `@EnableCaching` 어노테이션을 추가해야 한다.

```java
@EnableCaching
@EnableJpaAuditing
@SpringBootApplication
public class LearnredisApplication {

    public static void main(String[] args) {
        SpringApplication.run(LearnredisApplication.class, args);
    }
}
```

`@EnableCaching`은 Spring의 캐시 인프라를 활성화하며, 캐시 관련 어노테이션(`@Cacheable`, `@CachePut`, `@CacheEvict` 등)을 인식하고 처리할 수 있게 한다.

---

# 💼 캐싱 적용

캐싱을 적용할 부분은 다른 회원 프로필을 조회하는 로직이다. 이 서비스는 자주 조회되는 정보지만 자주 바뀌는 것은 아니기 때문에 여기에 캐싱을 하기로 했다. (참고로 OAuth2 프로젝트에는 없는 기능이다. Redis 학습을 위해 만든 API다.)

## 🔍 사용자 프로필 조회 (@Cacheable)

```java
@Cacheable(cacheNames = "userCache", key = "#id + ':profile'", cacheManager = "cacheManager")
@Transactional(readOnly = true)
public UserResponse getUserProfileById(Long id) {
    User findUser = userRepository.findById(id)
            .orElseThrow(() -> new CustomException(ErrorCode.USER_NOT_FOUND));
    return UserResponse.builder()
            .nickname(findUser.getNickname())
            .email(findUser.getEmail())
            .role(findUser.getRole().name())
            .picture(findUser.getPicture())
            .build();
}
```

`@Cacheable` 어노테이션의 의미:
- `cacheNames = "userCache"`: 'userCache'라는 이름의 캐시를 사용한다.
- `key = "#id + ':profile'"`: 캐시 키를 사용자 ID와 'profile' 문자열의 조합으로 생성한다.
- `cacheManager = "cacheManager"`: 앞서 정의한 캐시 매니저를 사용한다.

`@Cacheable`의 동작 방식:
1. 메서드가 호출될 때, 먼저 캐시에서 해당 키로 데이터를 찾는다.
2. 캐시에 데이터가 있으면, 메서드 실행 없이 캐시된 데이터를 즉시 반환한다.
3. 캐시에 데이터가 없으면, 메서드를 실행하고 그 결과를 캐시에 저장한 후 반환한다.

## ✏️ 사용자 프로필 업데이트 (@CacheEvict)

```java
@CacheEvict(cacheNames = "userCache", key = "#principal.getId() + ':profile'")
@Transactional
public UserResponse updateUserProfile(UserPrincipal principal, UserUpdateRequest request) {
    User findUser = userRepository.findById(principal.getId())
            .orElseThrow(() -> new CustomException(ErrorCode.USER_NOT_FOUND));

    User updateUser = findUser.updateProfile(request);

    return UserResponse.builder()
            .nickname(updateUser.getNickname())
            .email(updateUser.getEmail())
            .role(updateUser.getRole().name())
            .picture(updateUser.getPicture())
            .build();
}
```

`@CacheEvict` 어노테이션의 의미:
- `cacheNames = "userCache"`: 'userCache'에서 데이터를 제거한다.
- `key = "#principal.getId() + ':profile'"`: 제거할 캐시 항목의 키를 지정한다.

`@CacheEvict`의 동작 방식:
1. 메서드가 실행되기 전에 지정된 캐시에서 해당 키의 데이터를 제거한다.
2. 이후 메서드가 정상적으로 실행된다.
3. 결과적으로 사용자 프로필이 업데이트되면 기존의 캐시된 데이터는 삭제된다.

이를 통해 사용자가 프로필을 업데이트 하면 캐시에 저장된 데이터가 삭제되어 다음 조회 시 새로운 데이터가 캐시에 저장되어 정합성 문제를 해결할 수 있다.

---

# 📊 성능 비교

Redis 캐싱 적용 전후의 성능을 비교해 보았다. 
<br>

| 지표 | 캐싱 적용 전 | 캐싱 적용 후 | 변화율 |
|------|-------------|-------------|--------|
| 평균 응답 시간 (ms) | 3610 | 1918 | -46.9% |
| 최소 응답 시간 (ms) | 2 | 1 | -50% |
| 최대 응답 시간 (ms) | 15263 | 7130 | -53.3% |
| 표준 편차 (ms) | 2264.91 | 1290.19 | -43% |
| 에러율 (%) | 0.000 | 0.000 | 변화 없음 |
| 처리량 (요청/초) | 229.71 | 355.42 | +54.7% |
| 수신 데이터량 (KB/초) | 100.94 | 153.41 | +52% |
| 송신 데이터량 (KB/초) | 52.72 | 81.57 | +54.7% |
| 평균 응답 크기 (Bytes) | 450.0 | 442.0 | -1.8% |

**개선사항:**
1. **응답 시간 개선**: 평균 응답 시간이 46.9% 감소했다. 

2. **처리량 증가**: 초당 처리할 수 있는 요청 수가 54.7% 증가했다. 이는 서버의 전반적인 성능과 확장성이 향상되었음을 의미한다.

3. **일관성 향상**: 응답 시간의 표준 편차가 43% 감소했다. 이는 응답 시간의 일관성이 개선되어 사용자에게 더 안정적인 서비스를 제공할 수 있게 되었음을 나타낸다.

4. **리소스 효율성**: 평균 응답 크기가 약간 감소했지만, 처리량이 크게 증가했다. 이는 동일한 네트워크 리소스로 더 많은 요청을 처리할 수 있게 되었음을 의미한다.