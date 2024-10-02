---
title: "[Java] 리플렉션(Reflection)이란?"
excerpt: ""

categories:
  - Java
tags:
  - [Java]

permalink: /java/reflection

toc: true
toc_sticky: true

date: 2024-10-02
last_modified_at: 2024-10-02
---
# 🔍 리플렉션이란?

> reflection  
명사 (거울 등에 비친) 상\[모습]

리플렉션은 프로그램이 자기 자신을 들여다보고 분석할 수 있는 능력이다. 이것은 마치 사람이 거울을 보며 자신의 모습을 관찰하는 것과 비슷하다.

**정의:** 리플렉션은 실행 중인 Java 프로그램이 자신의 구조와 동작을 검사하고, 수정하거나 조작할 수 있게 해주는 Java의 Reflection API다.

**작동 원리:**
1. 클래스 로딩: JVM이 클래스를 로드할 때, 해당 클래스의 구조 정보를 메서드 영역에 저장한다.
2. Class 객체 생성: 각 로드된 클래스에 대해 Class 타입의 객체가 생성되어 힙 영역에 저장된다.
3. 리플렉션 API 사용: 프로그래머는 이 Class 객체를 통해 클래스의 구조 정보에 접근하고 조작한다.

---

# 🛠️ 어떤 경우에 사용할까?

컴파일 시점에는 어떤 타입의 클래스를 사용할지 모르지만, 런타임 시점에 가져와 실행해야 하는 경우 필요하다.  
리플렉션을 사용한 예시로는:

- Intellij의 자동완성
- 스프링의 어노테이션 처리
- ORM 프레임워크 (예: Hibernate)
- 단위 테스트 프레임워크 (예: JUnit)

---

# 💻 간단한 사용법 알아보기

Dog Class:

```java
public class Dog {
    public String name = "poppy";

    public Dog() {
    }
    
    public void run() {
        System.out.println(name + " run");
    }
}
```

사용 예시:

```java
public class DogMain {
    public static void main(String[] args) throws Exception {
        Dog dog = new Dog();
        Class<Dog> dogClass = Dog.class;

        // 클래스 이름 불러오기
        System.out.println("클래스명 -> " + dogClass.getName());
        // 클래스명 -> reflection.Dog

        // 생성자 불러오기
        System.out.println("생성자 -> " + dogClass.getDeclaredConstructor());
        // 생성자 -> public reflection.Dog()

        // 메서드 불러오기
        System.out.println("메서드 -> " + dogClass.getDeclaredMethod("run"));
        // 메서드 -> public void reflection.Dog.run()
        
        // 변수 불러오기
        System.out.println("변수명 -> " + dogClass.getDeclaredField("name"));
        // 변수명 -> public java.lang.String reflection.Dog.name

        // 변수의 값 불러오기 및 변경하기
        Field field = dogClass.getDeclaredField("name");
        
        System.out.println("변수값 -> " + field.get(dog));
        // 변수값 -> poppy
        
        field.set(dog, "chulsu");
        System.out.println("변수값 변경 -> " + field.get(dog));
        // 변수값 변경 -> chulsu
    }
}
```

---

# 🚧 리플렉션의 단점

- 성능 저하: 동적으로 클래스를 생성하기 때문에 JVM 컴파일러가 최적화할 수 있는 여지가 없다. 그러나 최신 JVM에서는 이러한 문제를 많이 개선했다.
- 보안 문제: `field.setAccessible(true)`를 설정하게 되면 private으로 설정해놓은 것도 직접 접근이 가능해지기 때문에 내부를 노출해서 추상화가 파괴된다.
- 코드 복잡성 증가: 가독성이 떨어지는 코드로 인해 가독성이 저하된다.

---

# 🍃 스프링 어노테이션과 리플렉션

스프링에서 어노테이션만으로도 수많은 기능이 동작하는 마법이 바로 이 리플렉션을 사용한 것이다.

```java
// 스프링의 어떤 클래스 내부의 메서드 중 하나
public <T extends Annotation> T getAnnotation(Class<T> annotationClass) {
    for (Annotation annotation : getAnnotations()) {
        if (annotation.annotationType() == annotationClass) {
            return (T) annotation;
        }
    }
    return null;
}
```

위 코드와 같이 스프링에서는 리플렉션 객체를 인자로 받아서 다양한 작업을 수행한다.  
주요 과정은 다음과 같다:

1. 스프링이 실행되면 `@ComponentScan` 어노테이션이 지정한 패키지의 클래스들을 스캔한다.
2. 스캔하면서 어노테이션에 대한 정보를 모두 불러온다. (클래스 레벨 먼저 스캔하여 빈을 등록하고, 그 후 필드와 메서드를 처리한다.)
3. 어노테이션의 이름과 추가 정보(예: `value = "blahblah"`)에 맞춰 스프링에서 해당 기능을 연결한다.
4. 이러한 과정을 통해 `@Component`를 찾아 빈을 등록하고, `@GetMapping`과 같은 어노테이션이 붙은 메서드에 URL을 매핑해준다.

---

<p class='ref'>참고</p>
- [https://velog.io/@ych0716/reflection](https://velog.io/@ych0716/reflection){: target='_blank'}



