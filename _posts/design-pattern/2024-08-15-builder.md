---
title: "[Design Pattern] 빌더 패턴 (Builder Pattern)"
excerpt: ""

categories:
  - Design Pattern
tags:
  - [Design Pattern]

permalink: /design-pattern/builder/

toc: true
toc_sticky: true

date: 2024-08-15
last_modified_at: 2024-08-15
---
# 🏢 Builder Pattern
빌더 패턴(Builder)은 복잡한 객체의 생성 과정과 표현 방법을 분리하여 다양한 구성의 인스턴스를 만드는 **생성 패턴**이다. 생성자에 들어갈 매개 변수를 메서드로 하나하나 받아들이고 마지막에 통합 빌드해서 객체를 생성하는 방식이다.

---

## 📈 빌더 패턴이 해결하고자 하는 문제
빌더 패턴은 생성과 관련된 어떤 문제를 해결하려고 했을가?

결론부터 말하자면, 객체를 생성할 때 **생성자(Constructor)만 사용할 때 발생할 수 있는 문제를 개선**하기 위해 고안됐다.

빌더 패턴을 다른 디자인 패턴과 비교해보면서, 코드 예제를 통해 이해해보는 시간을 가져보자.

### 🎲 점층적 생성자 패턴
점층적 생성자 패턴을 적용하면 생성자 [오버로딩](){: target="_blank"}을 통해 구현할 수 있다.

```java
public class People {
  private int age;
  private String name; // 필수 매개변수
  private String address;
  private String job;

  public People(int age, String name, String address, String job) {
    this.age = age;
    this.name = name;
    this.address = address;
    this.job = job;
  }

  public People(int age, String name, String address) {
    this.age = age;
    this.name = name;
    this.address = address;
  }

  ...
}
```
```java
public static void main(String[] args) {
    // 모든 정보가 있는 사람
    People people1 = new People(20, "미미", "서울시 강남구", "개발자");

    // 이름과 직업만 있는 사람
    People people2 = new People(0, "철수", null, "기획자");
}
```
이러한 방식은 클래스 인스턴스 필드들이 많을 수록 생성자에 들어갈 인자의 수가 늘어나 몇번째 인자가 어떤 필드였는지 헷갈릴 경우가 생기게 된다.(물론 인텔리제이 사용하면 친절하게 어떤 필드인지 알려준다.) 

또한 위의 예시들과 같이 필요하지 않는 값에는 null이나 0을 억지로 채워넣어야 한다.

그리고 무엇보다 타입이 다양할 수록 생성자 메서드 수가 기하급수적으로 늘어나 가독성이나 유지보수 측면에서 좋지 않다.

### 🎮 자바 빈(Bean) 패턴
이러한 단점을 보완하기 위해 setter 메소드를 사용한 자바 빈(Bean) 패턴이 고안됐다.

```java
public class People {
  private int age;
  private String name;
  private String address;
  private String job;

  public People() {}

  // Setter ...
}
```
```java
public static void main(String[] args) {
    // 모든 정보가 있는 사람
    People people1 = new People();
    people1.setAge(20);
    people1.setName("미미");
    people1.setAddress("서울시 강남구");
    people1.setJob("개발자");

    // 이름과 직업만 있는 사람
    People people2 = new People();
    people2.setName("철수");
    people2.setJob("기획자");
}
```
기존 생성자 오버로딩에서 나타났던 가독성 문제점이 사라지고 선택적인 파라미터에 대해 해당되는 Setter 메서드를 호출함으로써 유연적으로 객체 생성이 가능해졌다. 하지만 이러한 방식은 객체 생성 시점에 모든 값들을 주입 하지 않아 **일관성(consistency)** 문제와 **불변성(immutable)** 문제가 나타나게 된다.

**일관성 문제**  
필수 매개변수란 객체가 초기화될때 반드시 설정되어야 하는 값이다. 하지만 개발자가 깜빡하고 `setName()` 메서드를 호출하지 않았다면 이 객체는 일관성이 무너진 상태가 된다. 즉, 객체가 유효하지 않은 것이다. 만일 다른곳에서 사람 인스턴스를 사용하게 된다면 런타임 예외가 발생할 수도 있다.

이는 객체를 생성하는 부분과 값을 설정하는 부분이 물리적으로 떨어져 있어서 발생하는 문제점이다. 물론 이는 어느정도 생성자(Constructor)와 결합하여 극복은 할 수 있다. 하지만 다음에 소개할 불변성의 문제 때문에 자바 빈즈 패턴은 지양해야 한다.

**불변성 문제**  
자바 빈즈 패턴의 Setter 메서드는 객체를 처음 생성할때 필드값을 설정하기 위해 존재하는 메서드이다. 하지만 객체를 생성했음에도 여전히 외부적으로 Setter 메소드를 노출하고 있으므로, 협업 과정에서 언제 어디서 누군가 Setter 메서드를 호출해 함부로 객체를 조작할수 있게 된다. 이것을 불변함을 보장할 수 없다고 얘기한다.

### 🧸 드디어 빌더 패턴!
빌더 패턴은 이러한 문제들을 해결하기 위해 **별도의 Builder 클래스를 만들어** 필수 값에 대해서는 생성자를 통해, 선택적인 값들에 대해서는 메소드를 통해 **step-by-step**으로 값을 입력받은 후에 `build()` 메소드를 통해 최종적으로 하나의 인스턴스를 생성하여 리턴하는 패턴이다.

```java
public static void main(String[] args) {
    
    // 생성자 방식
    People people1 = new People(0, "미미", null, "개발자");

    // 빌더 방식
    People people = new People.Builder(20)
        .name("미미")
        .job("개발자")
        .build();
}
```
빌더 패턴을 이용하면 더이상 생성자 오버로딩 열거를 하지 않아도 되며, 데이터의 순서에 상관없이 객체를 만들어내 생성자 인자 순서를 파악할 필요도 없고 잘못된 값을 넣는 실수도 하지 않게 된다. 점층적 생성자 패턴과 자바빈즈 패턴 두 가지의 장점만을 취하였다고 볼 수 있다.

---
# 🌶️ Lombok의 @Builder
롬복에서는 개발자가 좀더 편하게 빌더 패턴을 이용하기 위해 별도의 어노테이션을 지원한다.

## 🌵 Lombok의 @Builder 사용시 주의사항
클래스위에 `@Builder` 어노테이션을 작성할 경우 모든 멤버 변수를 받는 기본 생성자를 만들고 그에 맞춰 빌더 패턴 코드가 완성된다. id, createdAt 같은 DB가 자동적으로 생성해주는 값들을 지정할 수 있도록 열려있는 구조는 문제가 될 여지가 다분하다.

**따라서 작업에 필요한 매개변수를 설정한 생성자 위에 @Builder 어노테이션을 붙여서 사용하자.**  
생성자 상단에 선언 시 생성자에 포함된 필드만 빌더에 포함된다.
```java
public class Post {
   private Long id;
   private String title;
   private String Content;
   private String author;
   private LocalDateTime createdAt;

   @Builder
   public Post(String title, String content, String author) {
      this.title = title;
      this.content = content;
      this.author = author;
   }
}
```

---

# 🗿 빌더 패턴 장단점
## 👍 장점
## 👎 단점

---

<p class="ref">참고</p>
- [https://inpa.tistory.com/entry/-💠-빌더Builder-패턴-끝판왕-정리](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EB%B9%8C%EB%8D%94Builder-%ED%8C%A8%ED%84%B4-%EB%81%9D%ED%8C%90%EC%99%95-%EC%A0%95%EB%A6%AC){: target="_blank"}
- [https://dev-youngjun.tistory.com/197](https://dev-youngjun.tistory.com/197){: target="_blank"}
- [https://dalichoi.tistory.com/entry/Builder-패턴과-Lombok-Builder-사용-시-주의사항](https://dalichoi.tistory.com/entry/Builder-%ED%8C%A8%ED%84%B4%EA%B3%BC-Lombok-Builder-%EC%82%AC%EC%9A%A9-%EC%8B%9C-%EC%A3%BC%EC%9D%98%EC%82%AC%ED%95%AD){: target="_blank"}

