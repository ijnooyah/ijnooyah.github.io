---
title: "[Java] static 키워드 마스터 하기 "
excerpt: ""

categories:
  - Java
tags:
  - [Java]

permalink: /java/static/

toc: true
toc_sticky: true

date: 2024-08-23
last_modified_at: 2024-08-23
---
# 🧩 static란?
static은 '정적인, 고정된'이라는 뜻을 가지고 있다. 
이러한 이유는 바로 static 키워드가 붙은 변수나 메서드는 어떤 객체에 소속되는 것이 아닌, **클래스에 고정되어 있는 변수나 메서드**이기 때문이다.

static의 특징을 먼저 살펴보자.
- [메모리에 고정적으로 할당된다.](#-메모리에-고정적으로-할당)
- [객체 생성 없이 사용할 수 있다.](#-객체-생성-없이-사용-가능)
- [프로그램이 시작되면 메모리의 메서드 영역에 적재되고, 프로그램이 종료될 때까지 메모리에 유지된다](#-메서드-영역에-적재).
- [static 메서드 내에서는 인스턴스 변수를 사용할 수 없다](#-static-메서드-내-인스턴스-변수-사용-불가).

## 📦 메모리에 고정적으로 할당
static 키워드가 붙지 않은 메서드나 변수의 경우 객체가 생성될 때마다 호출되어 서로 다른 값을 가지고 있을 수 있다.  
그렇기 때문에 **모든 인스턴스에서 동일한 값을 유지해야 할 경우** static을 유용하게 사용할 수 있다.

## 🎛️ 객체 생성 없이 사용 가능
문자열을 "print: 문자열"과 같이 출력해주는 기능을 만들어보자.
```java
public class PrintUtil{
    
    public String print(String str) {
        return "print: " + str;
    } 
}
```
이를 사용해보자.
```java
public class PrintMain{
    
    public static void main(String[] args) {
        PrintUtil utils = new PrintUtil();
        String print = utils.deco("프린트합니다.");
    }
}
```
`print()`메서드를 호출하기 위해서는 `PrintUtil`의 인스턴스를 먼저 생성해야한다.  
그런데 우리는 `PrintUtil`의 `print()`라는 기능만 사용할 뿐 멤버 변수를 활용하는 작업도 하지 않았다.
이럴때 static 키워드를 이용해 객체 생성없이 `print()`메서드를 사용할 수 있다.
```java
public class PrintUtil{
    
    public static String print(String str) {
        return "print: " + str;
    } 
}
```
메서드 앞에 static 키워드를 붙였다. 이렇게 하면 정적 메서드를 만들 수 있다.
```java
public class PrintMain{
    
    public static void main(String[] args) {
        String print = PrintUtil.deco("프린트할 문자열");
    }
}
```
이 정적 메서드는 인스턴스 생성 없이 클래스 명을 통해서 바로 호출할 수 있다.  
이렇듯 static 키워드를 붙이면 객체 생성 없이도 메서드나 변수를 사용할 수 있다.

그렇다면 이게 어떻게 가능할까? 
## 🗂️ 메서드 영역에 적재
프로그램이 시작되어 클래스가 메모리에 올라가게 되면 static 키워드가 붙은 변수나 메서드는 클래스와 함께 자동으로 메모리의 [Method 영역](https://ijnooyah.github.io/java/jvm/#-method-영역){: target="_blank"}에 생성된다.  
자동으로 메모리에 올라가기 때문에 객체를 생성할 필요없이 바로 사용이 가능한 것이다.

일반적인 메서드와 인스턴스 변수는 객체를 생성하면 메모리의 [Heap 영역](https://ijnooyah.github.io/java/jvm/#%EF%B8%8F-heap){: target="_blank"}에 할당된다. 이 Heap 영역은 GC에 의해 자동으로 관리된다. 즉, 사용하지 않는 객체의 경우 알아서 삭제시킴으로써 메모리를 관리해준다.
반면, static 메서드와 변수는 Method 영역에 존재하기 때문에 이러한 관리를 받지 못한다. 
이로 인해 프로그램이 종료될 때까지 메모리에 유지된다. 따라서 static 사용은 메모리 관리 측면에서 주의해야한다. 과도한 사용은 메모리 낭비, Thread-safe 문제 등을 야기할 수 있다.

## 🔒 static 메서드 내 인스턴스 변수 사용 불가
static 메서드는 프로그램 실행과 동시에 메모리에 올라가기 때문에 인스턴스 변수는 사용할 수 없다. 인스턴스 변수는 객체를 생성해야만 사용이 가능하기 때문에 객체를 생성하기 전에 먼저 메모리에 올라가는 static 메서드에서는 사용할 수 없는 것이다.

# ⏳ 변수와 생명주기
- **지역 변수(매개변수 포함)**: 지역 변수는 스택 영역에 있는 스택 프레임 안에 보관된다. 메서드가 종료되면 스택 프레임도 제거 되는데 이때 해당 스택 프레임에 포함된 지역 변수도 함께 제거된다.
따라서 지역 변수는 생존 주기가 짧다.
- **인스턴스 변수**: 인스턴스에 있는 멤버 변수를 인스턴스 변수라 한다. 인스턴스 변수는 힙 영역을 사용한다. 힙 여역은 GC(가비지 컬렉션)의 대상이 되기 전까지는 생존하기 때문에 보통 지역 변수보다 생존 주기가 길다.
- **클래스 변수(static 변수)**: static이 붙은 멤버 변수를 말한다. 메서드 영역에 보관되는 변수이다. 메서드 영역은 프로그램 전체에서 사용하는 공용 공간이다. 클래스 변수는 해당 클래스가 JVM에 로딩 되는 순간 생성된다.
그리고 JVM이 종료될 때까지 생명주기가 이어진다. 따라서 가장 긴 생명주기를 가진다.

--- 

