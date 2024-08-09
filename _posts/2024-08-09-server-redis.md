---
title: "[Server] Redis가 도대체 뭐야?"
excerpt: "Redis에 대해 알아보기"

categories:
  - Server
tags:
  - [Server]

permalink: /server/redis/

toc: true
toc_sticky: true

date: 2024-08-09
last_modified_at: 2024-08-09
---
## 🚗 Redis
### 1. Redis란?
**Re**mote(원격) **Di**ctionary(key-value) **S**erver(서버)  
Remote(원격)에 위치하고 프로세스로 존재하는 In-Memory 기반의 Dictionary(key-value) 구조 데이터 관리 Server 시스템이다.

#### 인메모리(In-Memory)란?
인메모리란 컴퓨터의 주기억장치인 **RAM**에 데이터를 올려서 사용하는 방법을 말한다.

**왜 메모리에 데이터를 올릴까?**   
이유는 명확하게도 속도 때문이다.  
SSD,HDD같은 저장공간에서 데이터를 가져오는 것보다 RAM에서 데이터를 가져오는 속도가 많게는 수백배 이상 빠르다.  
때문에 Redis는 **속도가 빠르다**는 장점을 가지고 있다.

**단점**  
Redis는 빠른 속도를 자랑하는 대신 치명적인 단점이 있다.
바로 용량으로 인해 데이터 유실이 발생할 수 있다는 점이다.

#### key-value란?
Redis는 key-value 형태의 NoSQL이다.

---

### 2. Redis 특징

---

참고
- [https://velog.io/@hope1213/Redis%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C](https://velog.io/@hope1213/Redis%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)
- [https://www.youtube.com/watch?v=tVZ15cCRAyE](https://www.youtube.com/watch?v=tVZ15cCRAyE)

