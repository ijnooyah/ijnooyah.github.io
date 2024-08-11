---
title: "[Architecture] 도메인 주도 설계(DDD)"
excerpt: "도메인 주도 설계에 대해 알아보기"

categories:
  - Architecture
tags:
  - [Architecture]

permalink: /architecture/ddd/

toc: true
toc_sticky: true

date: 2024-08-10
last_modified_at: 2024-08-10
---
## 🐌 도메인 주도 설계(Domain-Driven Design)
**D**omain-**D**riven **D**esign 줄여서 DDD라고 부르는 도메인 주도 설계는  
말그대로 **도메인을 중심**으로 설계해 나가는 방법이다.

### 도메인이란?
소프트웨어로 해결해야 할 **문제의 영역**을 말한다.  
예를 들어 고객이 원하는 상품을 어떻게 잘 제공할 것인가? 의 대한 문제는 커머스라는 도메인이 있다.  
커머스 도메인의 하위 도메인으로 판매자가 무엇을 판매할 것인가? 의 대한 문제는 상품이라는 도메인이 있다.

### 기존 개발의 문제점
- 데이터베이스 위주의 개발
  - 비즈니스 로직과 저장 기술이 강하게 결합 되어있는 구조
  - 성능을 데이터베이스에 의존하게 됨
- 트랜잭션 스크립트 패턴
  - 서비스 계층에서 비지니스 로직을 처리하는 방법
  - 서비스에 중복되는 코드가 계속 생겨날 수 있고 이러한 점이 유지보수를 어렵게 함

### DDD를 사용해야하는 이유
- 비지니스를 도메인 별로 나누어 설계를 하여 모듈간의 의존성은 최소화되고 응집성은 최대화됨 따라서 **유지보수에 용이**
- **보편적인(ubiquitous)** 언어를 사용하여 도메인 전문가와 개발자가 동일한 도메인 모델에 대해 **공통의 이해**를 구축하도록 도움

### 단점 
- 도메인의 복잡성을 그대로 반영하기 때문에 애플리케이션의 복잡성을 증가시킬 수 있음  
&rarr; 개발자는 도메인의 복잡성을 최대한 간결하게 모델링하는데 주의를 기울여야 함
- 과도한 도메인 주도는 설계의 유연성을 떨어뜨릴 수 있음  
&rarr; 따라서 개발자는 도메인 모델을 너무 고수하지 않고, 필요에 따라 완화시킬 필요가 있음.
- 이외에 높은 학습 곡선, 추상화의 어려움, 의존성 주입의 어려움 등이 존재

---

참고
- [https://yoonbing9.tistory.com/121](https://yoonbing9.tistory.com/121)
- [https://incheol-jung.gitbook.io/docs/q-and-a/architecture/ddd](https://incheol-jung.gitbook.io/docs/q-and-a/architecture/ddd)
- [https://velog.io/@sussa3007/Spring-DDD-Architecture-%EA%B8%B0%EC%B4%88](https://velog.io/@sussa3007/Spring-DDD-Architecture-%EA%B8%B0%EC%B4%88)
- [https://velog.io/@hope0206/DDD-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%A3%BC%EB%8F%84-%EC%84%A4%EA%B3%84-%EC%95%A0%EA%B7%B8%EB%A6%AC%EA%B1%B0%ED%8A%B8Aggregate-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0](https://velog.io/@hope0206/DDD-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%A3%BC%EB%8F%84-%EC%84%A4%EA%B3%84-%EC%95%A0%EA%B7%B8%EB%A6%AC%EA%B1%B0%ED%8A%B8Aggregate-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

