---
title: "[Java] 문자열의 일부를 추출하기"
excerpt: "substring"

categories:
  - Java
tags:
  - [Java]

permalink: /java/substring/

toc: true
toc_sticky: true

date: 2024-08-11
last_modified_at: 2024-08-11
---
## 특정 인덱스부터 끝까지 추출하기
```java
public String substring(int beginIndex) // beginIndex ~ 끝까지 추출
```
## 특정 인데스부터 특정 인덱스 이전까지 추출하기
```java
public String substring(int beginInde, int endIndex) // beginIndex ~ endInex 이전까지 추출
```
`endindex`는 반환 문자열에 포함 안되는 점 주의하기
