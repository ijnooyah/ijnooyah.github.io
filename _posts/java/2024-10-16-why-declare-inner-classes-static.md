---
title: "[Java] 내부 클래스를 static으로 선언하는 이유"
excerpt: ""

categories:
  - Java
tags:
  - [Java]

permalink: /java/why-declare-inner-classes-static

toc: true
toc_sticky: true

date: 2024-10-16
last_modified_at: 2024-10-16
---

인텔리제이 IDE를 사용해 개발을 할 때 내부 클래스에 static을 선언하지 않았을 경우 static으로 선언하라는 경고 메세지를 본 적이 있을 것이다. 이번 포스팅에서는 왜 내부 클래스에 static 선언을 권장하는지에 대한 것을 알아보려 한다.

# ⚔️ 인스턴스 클래스 vs 정적 클래스

자바의 내부 클래스 종류는 다음과 같다.
1. 인스턴스 클래스 (non-static 내부 클래스)
2. 정적 클래스 (static 내부 클래스)
3. 지역 클래스 (메서드 내부에 정의된 클래스)
4. 익명 클래스

이 중에서 우리가 주목해야 하는 것은 인스턴스 클래스와 정적 클래스의 차이이다.

---
{: .style1}

## 🌋 인스턴스 내부 클래스의 문제점

인스턴스 내부 클래스는 외부 클래스의 인스턴스에 종속되어 있다. 이로 인해 몇 가지 중요한 문제가 발생할 수 있다.

#### 🕵️ 숨겨진 외부 참조
인스턴스 내부 클래스는 외부 클래스의 인스턴스에 대한 숨겨진 참조를 가진다. 이는 내부 클래스가 외부 클래스의 멤버를 사용하지 않더라도 발생한다. 

예를 들어, 다음과 같은 코드를 살펴보자:
```java
public class OuterClass {
    int o = 10;
    class Inner {
        int i = 20;
    }
}
```

![alt text](/assets/images/posts_img/java/why-declare-inner-class-static/bytecode.png)

이 코드를 컴파일하고 바이트 코드를 확인해 보면, `InnerClass`의 생성자에 `OuterClass`의 참조가 숨겨진 매개변수로 전달되는 것을 볼 수 있다.

#### 💧 메모리 누수
이러한 숨겨진 참조로 인해 심각한 메모리 누수 문제가 발생할 수 있다. 내부 클래스의 인스턴스가 살아있는 한, 외부 클래스의 인스턴스도 GC(가비지 컬렉션)의 대상이 되지 않는다. (외부 참조로 내부 클래스와 연결되어 있기 때문에)

```java
import java.util.ArrayList;

class OuterClass {
	// 외부 클래스 객체의 크기를 불리기 위한 배열 변수
    private int[] data;

    // 내부 클래스
    class InnerClass {
    }
	
    // 외부 클래스 생성자
    public OuterClass(int size) {
        data = new int[size]; // 사이즈를 받아 배열 필드의 크기를 불림
    }
	
    // 내부 클래스 객체를 생성하여 반환하는 메소드
    InnerClass getInnerObject() {
        return new InnerClass();
    }
}
```
```java
public class Main {
    public static void main(String[] args) {
    	// InnerClass 객체를 저장할 리스트
        ArrayList<Object> al = new ArrayList<>();
        
        for (int counter = 0; counter < 50; counter++) {
            // InnerClass 객체를 생성하기 위해 OuterClass를 초기화하고 메서드를 호출하여 리스트에 넣는다.
            // 이때 OuterClass 객체는 메소드 호출용으로 일회용으로 사용되고 버려지기 때문에 GC 대상이 되어야 한다.
            al.add(new OuterClass(100000000).getInnerObject());
            System.out.println(counter);
        }
    }
}
```
주석에도 적혀있듯이 메서드 호출용으로 일회용으로 사용된 OuterClass 객체는 더이상 필요없게 되어 GC 대상이 되어 힙 메모리에서 삭제되어야 할 것이다.  
하지만 이 코드를 실행 해보면 `OutOfMemoryError` 예외가 발생된다. 
원래라면 메서드 호출 용도로만 쓰여진 일회용 객체는 바로 GC 수거 대상이 되어 제거 되어야 하지만, **내부 클래스에서 외부 클래스를 참조하고 있는 관계**때문에 내부 클래스(`InnerClass`) 데이터가 살아있는 한 외부 클래스(`OuterClass`) 데이터도 계속 살아 있어 엄청난 데이터가 지속적으로 메모리에 쌓이게 되어 이러한 예외를 터뜨리게 되는 것이다.

---
{: .style1}

## 🌟 static 내부 클래스의 장점

정적 내부 클래스를 사용하면 이러한 문제를 해결할 수 있다.

#### 🔒 외부 참조 없음
static 내부 클래스는 외부 클래스의 인스턴스에 대한 참조를 갖지 않는다. 따라서 메모리 사용이 더 효율적이고, 외부 클래스의 인스턴스 없이도 내부 클래스를 사용할 수 있다.

#### 🛡️ 메모리 누수 방지
static 내부 클래스를 사용하면 위에서 본 메모리 누수 문제를 해결할 수 있다.

위에서 보았던 예제 코드에 static을 붙여보고 실행해보자.

```java
import java.util.ArrayList;

class OuterClass {
    private int[] data;

    // static 내부 클래스
    static class InnerClass {
    }
	
    public OuterClass(int size) {
        data = new int[size]; 
    }
	
    InnerClass getInnerObject() {
        return new InnerClass();
    }
}

public class Main {
    public static void main(String[] args) {
    
        ArrayList<Object> al = new ArrayList<>();
        for (int counter = 0; counter < 50; counter++) {
            al.add(new OuterClass(100000000).getInnerObject());
            System.out.println(counter);
        }
    }
}
```

해당 코드를 실행해보면 인스턴스 내부 클래스에서 발생하던 `OutOfMemoryError`없이 코드가 잘 실행되는 것을 확인 할 수 있다.
`InnerClass`가 `OuterClass`에 대한 참조를 갖지 않기 때문에, 더 이상 필요 없는 `OuterClass` 인스턴스들이 적절히 GC 대상이될 수 있는 것이다.

---

# 🎓 결론
1. 내부 클래스가 외부 클래스의 인스턴스 멤버에 접근할 필요가 없다면, 반드시 <mark>`static`으로 선언하자</mark>.
2. static 내부 클래스를 사용하면 메모리 사용이 더 효율적이고 잠재적인 메모리 누수를 방지할 수 있다.


---
<p class='ref'>참고</p>
- [https://velog.io/@dkajffkem/%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%99%9C-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%A0%EA%B9%8C](https://velog.io/@dkajffkem/%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-%EC%99%9C-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%A0%EA%B9%8C){: target='_blank'}
- [https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94%EC%9D%98-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%98%EC%9E%90](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%9E%90%EB%B0%94%EC%9D%98-%EB%82%B4%EB%B6%80-%ED%81%B4%EB%9E%98%EC%8A%A4%EB%8A%94-static-%EC%9C%BC%EB%A1%9C-%EC%84%A0%EC%96%B8%ED%95%98%EC%9E%90){: target='_blank'}