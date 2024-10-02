---
title: "[Spring Framework] 어노테이션"
excerpt: ""

categories:
  - Spring Framework
tags:
  - [Spring Framework]

permalink: /spring-framework/annotation

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---

# 🏷️ 메타데이터란?

어노테이션을 이해하기 위해서는 먼저 메타데이터의 개념을 알아야 한다.

메타데이터(Metadata)는 간단히 말해 '데이터에 대한 데이터'이다. 즉, 어떤 데이터의 속성, 특성, 구조 등을 설명하는 데이터를 말한다. 

예를 들어, 디지털 사진을 생각해자:
- 사진 자체는 주된 데이터다.
- 그 사진의 촬영 날짜, 카메라 모델, 노출 시간, GPS 좌표 등의 정보는 메타데이터다.

프로그래밍에서 메타데이터는 코드에 대한 추가 정보를 제공한다. 이 정보는 컴파일러, 개발 도구, 프레임워크 등에 의해 사용될 수 있다.

Java에서 메타데이터는 주로 다음과 같은 형태로 나타난다:

- 주석(Comments): 개발자가 코드에 대한 설명을 남기는 가장 기본적인 형태의 메타데이터다.
- Javadoc: 코드의 문서화를 위한 특별한 형식의 주석이다.
- 어노테이션(Annotations): Java 5부터 도입된, 코드에 직접 붙이는 메타데이터 형식이다.

어노테이션은 이 중에서 가장 구조화되고 기계가 읽기 쉬운 형태의 메타데이터이다. 어노테이션을 통해 우리는 코드의 동작을 변경하거나, 컴파일러에게 특정 지시를 내리거나, 런타임에 특정 기능을 수행하도록 할 수 있다.

---

# 📌 어노테이션이란?

어노테이션(Annotation)은 Java 5부터 도입된 기능으로, 소스 코드에 메타데이터를 추가할 수 있게 해주는 강력한 도구다. 위키백과에서는 다음과 같이 정의하고 있다:

> 자바 애너테이션(Java Annotation)은 자바 소스 코드에 추가하여 사용할 수 있는 메타데이터의 일종이다. 보통 @ 기호를 앞에 붙여서 사용한다. JDK 1.5 버전 이상에서 사용 가능하다. 자바 애너테이션은 클래스 파일에 임베디드되어 컴파일러에 의해 생성된 후 자바 가상머신에 포함되어 작동한다.

이 정의가 조금 어렵게 느껴질 수 있다. 그래서 우리는 이를 더 쉽게 이해해 보겠다.

---
{: .style1}

## 📜 어노테이션의 역사와 목적

과거에는 Java 코드와 관련 설정 파일을 따로 관리했다. 예를 들어, 버전 관리를 "ver @.@"와 같은 형식으로 구분했다. 이런 방식은 두 가지 주요 문제를 야기했다:

1. 개발자들이 Java 코드는 변경하면서 설정 파일 업데이트를 잊는 경우가 많았다.
2. 설정과 코드가 분리되어 있어 전체 시스템을 이해하고 개발하는 데 어려움이 있었다.

이러한 문제를 해결하기 위해 어노테이션이 도입되었다. 어노테이션을 사용하면 코드와 설정을 한 곳에서 관리할 수 있게 된것이다.

---
{: .style1}

## 🧱 어노테이션의 기본 구조

어노테이션은 '@' 기호로 시작하며, 다음과 같은 기본 구조를 가진다:

```java
@AnnotationName(element = value, anotherElement = anotherValue)
```

여기서 `AnnotationName`은 어노테이션의 이름이고, 괄호 안의 내용은 어노테이션의 요소(element)와 그 값이다.

---

# 🌳 어노테이션의 종류

Java에서 어노테이션은 크게 세 가지 종류로 나눌 수 있다:

1. 표준(내장) 어노테이션
2. 메타 어노테이션
3. 사용자 정의 어노테이션

각각에 대해 자세히 알아보겠다.

---
{: .style1}

## 🏠 표준(내장) 어노테이션

Java에서 기본적으로 제공하는 어노테이션들이다. 가장 흔히 사용되는 몇 가지를 살펴보겠다.

### 🔄 @Override

이 어노테이션은 메서드가 상위 클래스의 메서드를 오버라이드(재정의)한다는 것을 명시적으로 선언한다.

```java
class Parent {
    void parentMethod() {}
}

class Child extends Parent {
    @Override
    void parentMethod() {
        // 부모 클래스의 메서드를 오버라이드
    }
}
```

`@Override`를 사용하면 두 가지 큰 이점이 있다:

1. 코드의 가독성이 향상된다. 다른 개발자가 코드를 볼 때 이 메서드가 오버라이드되었다는 것을 쉽게 알 수 있다.
2. 컴파일러가 오버라이딩을 검증한다. 만약 부모 클래스에 해당 메서드가 없다면 컴파일 에러가 발생한다.

---
{: .style1}

### 🚫 @Deprecated

이 어노테이션은 해당 요소(메서드, 클래스 등)가 더 이상 사용을 권장하지 않는다는 것을 나타낸다.

```java
public class OldClass {
    @Deprecated
    public void oldMethod() {
        // 이 메서드는 더 이상 사용을 권장하지 않음
    }
}
```

`@Deprecated`가 붙은 요소를 사용하면 컴파일러는 경고를 발생시킨다. 이는 개발자에게 해당 요소가 미래에 제거될 수 있음을 알려주는 역할을 한다.

자바의 하위 호환성 때문에 완전히 제거하지 않고 `@Deprecated`를 사용하는 경우가 많다. 예를 들어, `java.util.Date` 클래스의 `getDate()` 메서드는 다음과 같이 선언되어 있다:

```java
@Deprecated
public int getDate() {
    return normalize().getDayOfMonth();
}
```

---
{: .style1}

### 🔇 @SuppressWarnings

이 어노테이션은 컴파일러의 특정 경고 메시지를 억제한다.

```java
@SuppressWarnings("unchecked")
ArrayList list = new ArrayList(); // 제네릭 타입을 지정하지 않음
list.add(obj); // 원래 여기서 "unchecked" 경고가 발생하지만, @SuppressWarnings로 인해 억제됨
```

`@SuppressWarnings`는 다음과 같은 상황에서 유용하다:
- 레거시 코드와 작업할 때
- 경고를 발생시키는 코드가 실제로는 안전하다고 확신할 때
- 다른 중요한 경고들을 쉽게 식별하기 위해 안전한 경고들을 숨기고 싶을 때

하지만 무분별한 사용은 피해야 한다. 경고를 억제하기 전에 항상 그 경고의 원인을 이해하고 해결할 수 있는지 먼저 고려해야 한다.

---
{: .style1}

### 🔠 @FunctionalInterface

Java 8에서 도입된 이 어노테이션은 해당 인터페이스가 함수형 인터페이스임을 명시합니다. 함수형 인터페이스는 단 하나의 추상 메서드만을 가져야 한다.

```java
@FunctionalInterface
public interface SimpleFunction {
    void doSomething();
    
    // 다음과 같은 두 번째 추상 메서드를 추가하면 컴파일 에러가 발생한다.
    // void doSomethingElse();
}
```

`@FunctionalInterface`의 주요 목적은 다음과 같다:
1. 개발자의 의도를 명확히 표현한다. 이 인터페이스가 람다 표현식으로 사용될 수 있음을 나타낸다.
2. 컴파일러가 해당 인터페이스가 정확히 하나의 추상 메서드만 가지고 있는지 확인한다.

---
{: .style1}

## 🔍 메타 어노테이션

메타 어노테이션은 다른 어노테이션을 위한 어노테이션이다. 주로 커스텀 어노테이션을 만들 때 사용된다.

### 🎯 @Target

이 어노테이션은 새로운 어노테이션을 어디에 적용할 수 있는지 지정한다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target({ElementType.TYPE, ElementType.METHOD})
public @interface MyCustomAnnotation {
    // ...
}
```

위 예제에서 `@MyCustomAnnotation`은 클래스가, 인터페이스, enum(TYPE)과 메서드에만 사용할 수 있다.

주요 ElementType 값들:
- TYPE: 클래스, 인터페이스, enum
- FIELD: 필드
- METHOD: 메서드
- PARAMETER: 메서드 파라미터
- CONSTRUCTOR: 생성자
- LOCAL_VARIABLE: 지역 변수

---
{: .style1}

### ⏳ @Retention

이 어노테이션은 새로운 어노테이션이 얼마나 오래 유지될지를 지정한다.

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomAnnotation {
    // ...
}
```

RetentionPolicy의 주요 값들:
- SOURCE: 소스 코드에만 존재하며, 컴파일러에 의해 무시된다.
- CLASS: 클래스 파일에 존재하지만, 런타임에는 사용할 수 없다. (기본값)
- RUNTIME: 클래스 파일에 존재하며, 런타임에 사용할 수 있다.

---
{: .style1}

### 📄 @Documented

이 어노테이션은 새로운 어노테이션이 Javadoc에 포함되어야 함을 나타낸다.

```java
import java.lang.annotation.Documented;

@Documented
public @interface MyCustomAnnotation {
    // ...
}
```

---
{: .style1}

### 👪 @Inherited

이 어노테이션은 새로운 어노테이션이 하위 클래스에 상속되어야 함을 나타낸다.

```java
import java.lang.annotation.Inherited;

@Inherited
public @interface MyCustomAnnotation {
    // ...
}

@MyCustomAnnotation
class Parent {}

class Child extends Parent {} // Child 클래스도 @MyCustomAnnotation을 상속받음
```

---
{: .style1}

### 🔁 @Repeatable

Java 8에서 도입된 이 어노테이션은 새로운 어노테이션을 같은 요소에 여러 번 적용할 수 있게 한다.
```java
import java.lang.annotation.Repeatable;

@Repeatable(Schedules.class)
@interface Schedule {
    String dayOfMonth() default "first";
    String dayOfWeek() default "Mon";
    int hour() default 12;
}

@interface Schedules {
    Schedule[] value();
}

class MeetingRoom {
    @Schedule(dayOfMonth="last")
    @Schedule(dayOfWeek="Fri", hour=23)
    public void scheduleMeetings() { ... }
}
```

`@Repeatable`을 사용할 때는 컨테이너 어노테이션(위 예제의 `Schedules`)도 함께 정의해야 한다.

---

## 🛠️ 사용자 정의 어노테이션

개발자가 직접 정의하는 어노테이션이다. 기본 구조는 다음과 같다:

```java
public @interface MyAnnotation {
    String value() default "defaultValue";
    int count() default 1;
}
```

사용 예:

```java
@MyAnnotation(value = "custom", count = 3)
public void myMethod() {
    // ...
}
```

사용자 정의 어노테이션을 만들 때 주의할 점들:

1. 요소의 타입은 기본형, String, Class, enum, 어노테이션, 또는 이들의 배열만 가능하다.
2. 괄호() 안에 매개변수를 선언할 수 없다.
3. 예외를 선언할 수 없다.
4. 요소를 타입 매개변수로 정의할 수 없다.

잘못된 예:

```java
public @interface InvalidAnnotation {
    int id = 100; // 상수 선언은 가능
    String major(int i, int j); // 매개변수 선언 불가
    String minor() throws Exception; // 예외 선언 불가
    ArrayList<T> list(); // 타입 매개변수 사용 불가
}
```

---

# ⚙️ 어노테이션 처리

어노테이션은 그 자체로는 아무 동작도 하지 않는다. 어노테이션의 정보를 읽고 처리하는 것은 컴파일러나 런타임 환경이다.

## ⏱️ 컴파일 타임 처리

컴파일 타임에 어노테이션을 처리하려면 어노테이션 프로세서(Annotation Processor)를 사용해야 한다. 이는 javac의 플러그인으로 동작하며, 소스 코드를 분석하고 처리한다.

어노테이션 프로세서를 만드는 기본 단계는 다음과 같다:

1. `javax.annotation.processing.AbstractProcessor` 클래스를 상속받는다.
2. `@SupportedAnnotationTypes`와 `@SupportedSourceVersion` 어노테이션을 사용하여 처리할 어노테이션과 소스 버전을 지정한다.
3. `process()` 메서드를 오버라이드하여 어노테이션 처리 로직을 구현한다.

예를 들어, 간단한 로그 메시지를 생성하는 어노테이션 프로세서를 만들어 보겠다:

```java
import javax.annotation.processing.*;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.*;
import java.util.Set;

@SupportedAnnotationTypes("com.example.LogMe")
@SupportedSourceVersion(SourceVersion.RELEASE_8)
public class LogProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (TypeElement annotation : annotations) {
            Set<? extends Element> annotatedElements = roundEnv.getElementsAnnotatedWith(annotation);
            for (Element element : annotatedElements) {
                System.out.println("Found @LogMe at " + element.toString());
            }
        }
        return true;
    }
}
```

이 프로세서는 `@LogMe` 어노테이션이 사용된 모든 요소를 찾아 로그 메시지를 출력한다.

---
{: .style1}

## 🏃 런타임 처리

런타임에 어노테이션을 처리하려면 리플렉션(Reflection) API를 사용한다. 이를 통해 프로그램 실행 중에 클래스, 메서드, 필드 등의 메타데이터를 검사할 수 있다.

> **리플렉션 API란?**  
리플렉션은 실행 중인 자바 프로그램이 자기 자신을 검사하거나 내부 속성을 조작할 수 있게 해주는 기능이다. 이를 통해 프로그램은 컴파일 시점에 알려지지 않은 클래스를 사요할 수 있고, 그 클래스의 메서드, 필드, 생성자 등을 탐색할 수 있다.  
자세한 내용은 [해당 글](https://ijnooyah.github.io/java/reflection)을 참고하자.
{: .info}

예를 들어, 메서드 실행 시간을 측정하는 `@Timed` 어노테이션을 만들고 처리해보겠다:

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface Timed {}
```

이 어노테이션을 처리하는 코드는 다음과 같다:

```java
import java.lang.reflect.Method;

public class TimedMethodInvoker {
    public static void invokeTimedMethod(Object obj, String methodName) throws Exception {
        Method method = obj.getClass().getMethod(methodName);
        if (method.isAnnotationPresent(Timed.class)) {
            long start = System.currentTimeMillis();
            method.invoke(obj);
            long end = System.currentTimeMillis();
            System.out.println(methodName + " took " + (end - start) + " ms");
        } else {
            method.invoke(obj);
        }
    }
}
```

이 코드는 주어진 객체의 메서드가 `@Timed` 어노테이션을 가지고 있는지 확인하고, 있다면 실행 시간을 측정한다.

---

# 💼 어노테이션의 실제 활용 사례

어노테이션은 다양한 프레임워크와 라이브러리에서 광범위하게 사용된다. 몇 가지 대표적인 예를 살펴보겠다.

## 🍃 Spring Framework

Spring Framework는 어노테이션을 광범위하게 사용하는 대표적인 예이다.

- `@Controller`, `@Service`, `@Repository`: 각각 MVC 패턴의 컨트롤러, 서비스 계층, 데이터 접근 계층을 나타낸다.
- `@Autowired`: 의존성 주입을 위해 사용된다.
- `@RequestMapping`: HTTP 요청을 특정 핸들러 메서드에 매핑한다.

예시:

```java
@Controller
public class HelloController {

    @Autowired
    private HelloService helloService;

    @RequestMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("message", helloService.getHelloMessage());
        return "hello";
    }
}
```

---
{: .style1}

## 💾 JPA(Java Persistence API)

JPA는 Java 애플리케이션에서 관계형 데이터를 관리하기 위한 명세로, 어노테이션을 사용하여 객체와 데이터베이스 테이블 간의 매핑을 정의한다.

- `@Entity`: 클래스가 데이터베이스 테이블에 매핑됨을 나타낸다.
- `@Id`: 엔티티의 기본 키를 지정한다.
- `@Column`: 필드와 데이터베이스 컬럼 간의 매핑을 세부적으로 지정한다.

예시:

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false)
    private String username;

    @Column(nullable = false)
    private String email;

    // getters and setters
}
```

---
{: .style1}

## 🦋 Lombok

Lombok은 Java의 상용구 코드를 줄이는데 도움을 주는 라이브러리다. 어노테이션을 사용하여 컴파일 시점에 코드를 생성한다.

- `@Getter`, `@Setter`: 자동으로 getter와 setter 메서드를 생성한다.
- `@ToString`: `toString()` 메서드를 자동으로 생성한다.
- `@EqualsAndHashCode`: `equals()`와 `hashCode()` 메서드를 자동으로 생성한다.

예시:

```java
import lombok.Data;

@Data
public class User {
    private Long id;
    private String username;
    private String email;
}
```

이 간단한 클래스에 `@Data` 어노테이션을 사용함으로써, Lombok은 모든 필드에 대한 getter와 setter, `toString()`, `equals()`, `hashCode()` 메서드를 자동으로 생성한다.

---

# 🚧 어노테이션 사용 시 주의사항

어노테이션은 강력한 도구이지만, 남용하면 코드의 가독성과 유지보수성을 해칠 수 있다.  
다음은 어노테이션 사용 시 주의해야 할 몇 가지 사항이다:

1. **과도한 사용 지양**: 어노테이션이 너무 많으면 코드 읽기가 어려워질 수 있다. 꼭 필요한 경우에만 사용하자.

2. **문서화**: 커스텀 어노테이션을 만들 때는 반드시 그 목적과 사용법을 문서화하자. `@Documented` 어노테이션을 사용하면 Javadoc에 자동으로 포함된다.

3. **성능 고려**: 특히 런타임에 처리되는 어노테이션의 경우, 과도한 사용은 성능에 영향을 줄 수 있다. 필요한 경우에만 `RetentionPolicy.RUNTIME`을 사용하자.

4. **버전 호환성**: 어노테이션 사용 시 Java 버전 호환성을 고려해야 한다. 일부 어노테이션은 특정 Java 버전 이상에서만 사용 가능하다.

5. **테스트**: 어노테이션 프로세서를 만들 때는 다양한 상황에 대한 테스트를 철저히 해야 한다. 컴파일 시점 에러는 디버깅하기 어려울 수 있다.

---
<p class='ref'>참고</p>
- [https://velog.io/@jkijki12/annotation](https://velog.io/@jkijki12/annotation){: target='_blank'}