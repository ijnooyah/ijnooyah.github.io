---
title: "[Java] @PostConstruct"
excerpt: ""

categories:
  - Java
tags:
  - [Java]

permalink: /java/postconstruct

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---

# 📌 개요

`@PostConstruct`는 의존성 주입(Dependency Injection)이 완료된 후 초기화를 수행하기 위해 사용된다.

---

# 🔑 주요 특징

- **실행 시점**: 의존성 주입이 완료된 직후, 빈(Bean)이 초기화될 때 자동으로 실행된다.
- **호출 횟수**: 빈 생명주기 동안 단 한 번만 실행된다.
- **메소드 요구사항**: 
  - 반환 타입은 void여야 한다.
  - 매개변수를 가질 수 없다.
  - 예외를 던질 수 있다.

> **빈 생명 주기 (Bean LifeCycle)**  
스프링 컨테이너는 빈의 생성, 의존관계 주입, 초기화, 사용, 소멸의 과정을 관리한다.  
`@PostConstruct`는 의존관계 주입 완료 후 초기화 단계에서 실행된다. 이는 빈이 완전히 생성되고 사용 가능한 상태가 되는 시점을 정확히 포착하여 필요한 설정을 수행할 수 있게 해준다.

---

# ⏳ 실행 순서

1. 생성자 호출
2. 의존성 주입 완료
3. `@PostConstruct` 메소드 실행

---

# 💻 예제

```java
@Service
public class PostConstructTestService {

    @Autowired
    private TestMapper testMapper;

    public PostConstructTestService() {
        System.out.println("1. PostConstructTestService 생성자 호출");
        try {
            testMapper.getTest();
        } catch (NullPointerException e) {
            System.out.println("2. 생성자 내 testMapper는 null (DI 미완료)");
        }
    }
    
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct 메소드 호출");
        try {
            testMapper.getTest();
            System.out.println("4. testMapper 정상 작동 (DI 완료)");
        } catch (Exception e) {
            System.out.println("예외 발생: " + e.getMessage());
        }
    }
}
```

**실행 결과**
```
1. PostConstructTestService 생성자 호출
2. 생성자 내 testMapper는 null (DI 미완료)
3. @PostConstruct 메소드 호출
4. testMapper 정상 작동 (DI 완료)
```

1. **생성자 실행**: 
   - 객체가 생성될 때 가장 먼저 생성자가 호출된다.
   - 이 시점에서는 아직 의존성 주입이 완료되지 않았으므로 `testMapper`는 `null`이다.

2. **의존성 주입**:
   - 생성자 호출 후, 스프링 컨테이너가 `@Autowired`로 지정된 `testMapper`에 대한 의존성을 주입한다.

3. **@PostConstruct 메소드 실행**:
   - 의존성 주입이 완료된 후 `@PostConstruct`가 지정된 `init()` 메소드가 자동으로 호출된다.
   - 이 시점에서는 `testMapper`가 정상적으로 주입되어 있어 사용 가능하다.


---

<p class='ref'>참고</p>
- [https://velog.io/@limsubin/Spring-Boot%EC%97%90%EC%84%9C-PostConstruct-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90](https://velog.io/@limsubin/Spring-Boot%EC%97%90%EC%84%9C-PostConstruct-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90){: target='_blank'}
