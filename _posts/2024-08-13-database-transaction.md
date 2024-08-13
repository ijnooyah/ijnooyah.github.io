---
title: "[Database] 트랜잭션(Transaction)"
excerpt: "트랜잭션에 대해 알아보기"

categories:
  - Database
tags:
  - [Database]

permalink: /database/transaction/

toc: true
toc_sticky: true

date: 2024-08-13
last_modified_at: 2024-08-13
---
# 🐙 트랜잭션(Transaction)란?
트랜잭션(Transaction)이란 데이터베이스의 상태를 변화시키기 위해 수행하는 **작업의 단위**를 뜻한다.

DB의 상태를 변화시킨다는 것은 무엇을 뜻하는가?  
SELECT, UPDATE, INSERT, DELETE 와 같은 쿼리를 날려 연산을 수행하는 것이다.

여기서 착각하지 말하야할 것은 
작업의 단위는 SELECT나 UPDATE 등 SQL문 하나가 아니라는 점이다.  

```
예시) 사용자 A가 사용자 B에게 만원을 송금한다

1. 사용자 A의 계좌에서 만원을 차감한다: UPDATE를 통해 사용자 A의 잔고 변경
2. 사용자 B의 계좌에 만원을 추가한다: UPDATE를 통해 사용자 B의 잔고 변경

작업 단위: 출금 UPDATE + 입금 UPDATE  
→ 통틀어 하나의 트랜잭션이라고 한다.    
```  
- 두 쿼리문이 모두 성공적으로 완료되어야만 하나의 작업(트랜잭션)이 완료되는 것이다. (Commit)

- 작업 단위에 속하는 쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전 상태로 돌려놓아야한다. (Rollback)


---

참고
- [https://gyoogle.dev/blog/computer-science/data-base/Transaction.html](https://gyoogle.dev/blog/computer-science/data-base/Transaction.html)
- [https://mommoo.tistory.com/62](https://mommoo.tistory.com/62)

