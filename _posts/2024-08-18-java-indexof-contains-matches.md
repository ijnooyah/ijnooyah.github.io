---
title: "[Java] 문자열에 특정 문자 포함 여부 확인하기"
excerpt: "indexOf, contains, matches"

categories:
  - Java
tags:
  - [Java]

permalink: /java/indexof-contains-matches/

toc: true
toc_sticky: true

date: 2024-08-18
last_modified_at: 2024-08-18
---
# indexOf
```java
public int indexOf(int ch)
public int indexOf(int ch, int fromIndex)
public int indexOf(String str)
public int indexOf(String str, int fromIndex)
```
문자열에서 처음 나타나는 index를 찾아서 index의 위치를 리턴한다.  
만약, 문자열에서 찾는 문자나 문자열이 없다면 -1을 리턴

fromIndex 두번째 파라미터가 존재하면 문자열의 fromIndex부터 파라미터로 입력받은 문자열을 찾음

---

# contains
```java
public boolean contains(CharSequence s)
```

---

# matches
```java
public boolean matches(String regex)
```
정규식을 이용함.
문자열에 정규식과 일치하는 부분이 있는지 체크함.
