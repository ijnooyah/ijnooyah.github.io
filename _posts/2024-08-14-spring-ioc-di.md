---
title: "[Spring] IoC? DI?"
excerpt: ""

categories:
  - Spring
tags:
  - [Spring]

permalink: /spring/ioc-di/

toc: true
toc_sticky: true

date: 2024-08-14
last_modified_at: 2024-08-14
---
스프링을 공부하다 보면 IoC 컨테이너, DI에 대한 이야기를 많이 봤을 것이다. 이 개념들은 공부하다보면 계속 나오는 단어들이기 때문에 한번 제대로 짚고 가지 않으면 스프링에 대한 이해를 하기 힘들다. 
이번 포스팅에서는 IoC, DI 뿐만 아니라 POJO, AOP, PSA도 정리를 해봤다. 스프링에 대한 기초 지식을 학습해 보자.

# 🍃 Spring Framework
![스프링 삼각형](/assets/images/posts_img/spring/ioc-di/spring-triangle.png)

Spring Framework(이하 Spring)은 자바를 기반으로 만들어진 엔터프라이즈 경량급 프레임워크다. 다양한 라이브러리와 기능을 제공해 보다 쉽게 어플리케이션을 만들 수 있게 해주며, POJO와 3개의 핵심 개념(IoC / DI, AOP, PSA)으로 설명된다.

> **참고**: 엔터프라이즈급 개발은 기업을 대상으로 하는 개발을 말하며, 경량급의 뜻은 스프링 자체가 가볍거나 작은 규모라는 뜻이 아니다. 불필요한 기능들을 제외했다는 면에서 무겁지 않다는 뜻이다.

---

# ☕ POJO(Plain Old Java Object) 
**스프링은 POJO(Plain Old Java Object) 기반의 프레임워크이다.**  
Plain Old Java Object
- 오래된 방식의 순수한 자바 오브젝트 / 평범한 구식 자바 객체  

마틴 파울러는 스프링 등장 전 널리 사용되던 EJB처럼 복잡하고 제한적인 기술보다는 **자바의 단순 오브젝트를 이용해 비지니스 로직을 구현하는 편이 낫다**고 생각했다. 따라서 왜 사람들이 사용하기를 꺼려하는지 의문을 가졌고, 그는 EJB처럼 그럴싸한 이름이 없어서 사람들이 사용을 주저하는 것이라고 결론을 내렸다. 거기서 탄생한 용어가 바로 POJO(Plain Old Java Object)다. 

> **EJB(Enterprise Java Beans)**  
Sun사에서 엔터프라이즈 개발을 단순화하기위해 만들어낸 Java 스팩이다.  
스프링이 등장하기 이전에는 EJB가 자바 엔터프라이즈 애플리케이션 개발 시장을 독점하고 있었다. 코드들이 EJB 기술에 지나치게 종속되어야 한다는 치명적인 단점이 있었다.

**POJO의 조건**  
- 특정 규약(contract)에 종속되지 않는다.
	- Java 언어와 꼭 필요한 API 외에 종속되지 않는다.
- 특정 환경에 종속되지 않는다.
- 객체지향 원리에 충실해야 한다.
	- 필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트

**POJO의 장점**
- 간결하고 깔끔하게 코드를 작성할 수 있다.
- 간편하게 테스트를 할 수 있다. (특정 기술이나 환경에 종속적이지 않기 때문에)
- 객체지향적인 설계를 자유롭게 적용할 수 있다.

---

# 🥪 IoC(Inversion of Control) 와 DI(Depedency Injection)
**스프링은 IoC(Inversion of Control)와 DI(Depedency Injection)을 지원한다.**
## 🍅 IoC(Inversion of Control)
IoC(Inversion of Control: 제어의 역전)은 객체를 생성, 제거 하는 **제어권**이 개발자가 아닌 **프레임워크(Spring, IoC Container)** 에게 위임 되는 것을 말한다.

**IoC의 장점**
- 프로그램의 진행 흐름과 구체적인 구현을 분리시킬 수 있다.
- 개발자는 비즈니스 로직에 집중할 수 있다.
- 구현체 사이의 변경이 용이하다.
- 객체 간 의존성이 낮아진다.

---

## 🥬 DI(Depedency Injection)
DI(Dependency Injection: 의존성 주입)은 **객체간의 의존 관계**를 직접 생성하는 것이 아닌 **외부(Ioc Container)로 부터 주입**받아 동적으로 의존관계가 성립되는 것을 말한다.

의존성 주입 방법에는 3가지가 있다.
- **생성자 주입** 
- Setter 주입 
- 필드 주입  

스프링은 세가지 방법 중 **생성자 주입** 방식을 권장한다.  
세가지 방법에 대한 자세한 설명은 [생성자 주입](){: target="_blank"} 에서 확인해보자.

**DI의 장점**
- 의존성이 줄어든다. (변경에 덜 취약해진다.)
- 모의 객체를 주입할 수 있기 때문에 단위 테스트가 쉬워진다.
- 가독성이 높아진다.
- 재사용성이 높아진다.

---

## 🍞 IoC 와 DI의 예시

**Before**
```java
@Service
public class MemberService {
	private final MemberRepository memberRepository = new MemberRepository();
}
```
- 클래스 내부에서 의존하는 객체를 new 키워드를 통해 개발자가 직접 생성했다.

**After**
```java
@Service
public class MemberService {
	private final MemberRepository memberRepository;

	@Autowired
	public MemberService(MemberRepository memberRepository) {
		this.memberRepository = memberRepository;
	}
}
```
- 클래스 내부에서 객체를 개발자가 직접 생성하는 것이 아닌, 제어권을 스프링에게 위임하여 스프링에게 객체 생성 및 생명주기를 맡겼다. &rarr; 제어의 역전
- @Autowired를 통해 객체를 주입받았다. &rarr; 의존성 주입

---

# 🕵️ AOP(Aspect Oriented Programming)
**스프링은 AOP(Aspect Oriented Programming)을 지원한다.**  

![AOP](/assets/images/posts_img/spring/ioc-di/aop.png)

AOP(Aspect Oriented Programming: 관점 지향 프로그래밍)은 로깅, 보안, 트랜잭션과 같은 공통 기능을 분리하여 하나의 책임을 가지게 하는 프로그래밍 기법을 말한다.

**AOP 장점**

**AOP 예시**

---

# 🌷 PSA(Portable Sevice Abstraction)
**스프링은 PSA(Portable Sevice Abstraction)을 지원한다.**  

PSA(Portable Sevice Abstraction: 일관성 있는 서비스 추상화)은 환경의 변화와 관계없이 일관된 방식의 기술 접근 환경을 제공하는 추상화 구조를 말한다.

**PSA 장점**

**PSA 예시**


---

<p class="ref">참고</p>
- [https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-%EA%B7%B8%EA%B2%8C-%EB%AD%94%EB%8D%B0](https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-%EA%B7%B8%EA%B2%8C-%EB%AD%94%EB%8D%B0)
- [https://velog.io/@jummi10/definition-of-spring-framework-and-features#2-plain-old-java-objectpojo-%EA%B8%B0%EB%B0%98%EC%9D%98-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EC%9D%B4%EB%8B%A4](https://velog.io/@jummi10/definition-of-spring-framework-and-features#2-plain-old-java-objectpojo-%EA%B8%B0%EB%B0%98%EC%9D%98-%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC%EC%9D%B4%EB%8B%A4)
- [https://velog.io/@doridam1116/Spring%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B0%9C%EB%85%90-DI-IoC-AOP-](https://velog.io/@doridam1116/Spring%EC%9D%98-%ED%95%B5%EC%8B%AC-%EA%B0%9C%EB%85%90-DI-IoC-AOP-)
- [https://velog.io/@dyko/Spring-PSA](https://velog.io/@dyko/Spring-PSA)



