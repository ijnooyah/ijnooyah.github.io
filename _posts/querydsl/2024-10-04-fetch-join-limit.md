---
title: "[Querydsl] Fetch Join과 LIMIT절을 같이 쓰는 것은 주의해야 한다"
excerpt: ""

categories:
  - Querydsl
tags:
  - [Querydsl, 트러블 슈팅]

permalink: /querydsl/fetch-join-limit

toc: true
toc_sticky: true

date: 2024-10-04
last_modified_at: 2024-10-04
---

# 🔍 Fetch Join과 페이징의 문제점

[도메인 모델](https://ijnooyah.github.io/spring-data-jpa/role#-%EB%8F%84%EB%A9%94%EC%9D%B8-%EB%AA%A8%EB%8D%B8%EA%B3%BC-%ED%85%8C%EC%9D%B4%EB%B8%94){: target='_blank'}을 보면 `User`와 `UserRole`은 일대다 관계이고 `UserRole`과 `Role`은 다대일 관계이다.  

사용자 검색 기능을 만들때, 검색 결과인 사용자들에 대한 권한 정보도 가져와야 하니 패치 조인을 사용해 모든 관련 데이터를 한 번에 가져오는 방식을 선택했다. 이생각으로 코드를 다음과 같이 작성하였다. (참고로 커서 기반 페이징을 사용한다.)

```java
return queryFactory
                .selectFrom(user)
                .leftJoin(user.userRoles, userRole).fetchJoin()
                .leftJoin(userRole.role, role).fetchJoin()
                .where(applySearchType(condition.getSearchInput(), condition.getSearchType())
                        .and(hasRoles(condition.getRoles()))
                        .and(hasProviders(condition.getProviders()))
                        .and(goeStartDate(condition.getStartDate()))
                        .and(loeEndDate(condition.getEndDate()))
                        .and(isNotDeleted())
                        .and(userIdGt(condition.getCursorId()))
                )
                .limit(condition.getSize() + 1)
                .fetch();
```

하지만 코드 실행 후, 콘솔에서 확인한 쿼리에는 `LIMIT` 절이 누락된 채 전체 데이터를 풀 스캔하는 문제가 있었다.

코드에서 `LIMIT` 절을 설정했음에도 쿼리에는 적용되지 않았고, 페이징은 적용이 되는 상황이었지만 모든 데이터를 메모리로 가져와 페이징하는 상황이 발생했다. <mark>그 이유는 바로 패치 조인과 페이징을 함께 사용할 수 없다는 Hibernate의 특성 때문이다.</mark> 

---

# ❓ 왜 Fetch Join과 LIMIT을 같이 사용할 수 없을까?
Hibernate는 컬렉션 패치 조인과 페이징을 함께 사용할 경우, 데이터베이스 레벨에서 페이징을 하지 않고 애플리케이션 메모리에서 페이징을 수행하는 방식으로 동작한다. 
이는 Hibernate 내부의 `QueryTranslatorImpl.java` 파일에서 확인할 수 있다. 이 파일의 코드를 살펴보면, 다음과 같은 조건에서 `selection` 객체를 복사하여 `queryParametersToUse`로 사용하는 부분이 있다:
```java
if (hasLimit && containsCollectionFetches()) {
    // Fetch Join과 페이징 제한(Limit)이 함께 사용되는 경우
}

```
이때 `selection` 객체는 페이징에 필요한 `firstRow`, `maxRows` 등의 정보를 담고 있지만, Hibernate의 내부 동작 방식에 따라 이 정보들이 모두 **null** 값으로 처리된다.  
결과적으로 페이징을 위한 정보가 쿼리에 반영되지 않으므로, 쿼리 실행 시 `LIMIT` 절이 적용되지 않고 모든 데이터를 불러오는 문제가 발생하게 된다.

---

# 🚧 문제의 발생 과정
1. **쿼리 실행 시 경고 발생**
   경고 메세지:
   ```
    firstResult/maxResults specified with collection fetch; applying in memory!
   ```
   이는 페이징이 데이터베이스에서 처리되지 않고, 메모리에서 처리됨을 의미한다.

2. **메모리에서 페이징 처리**  
   예를 들어, 10개의 데이터를 요청했더라도 실제로는 모든 데이터를 읽어와 메모리에서 페이징을 처리한다.  
   이는 성능 문제를 일으킬 수 있다.

3. **최적화 실패**  
   애초에 성능을 고려한 패치 조인이었지만, 오히려 성능 저하를 초래하게 된다.

Hibernate 공식 문서에서도 패치 조인과 페이징을 함께 사용하는 것에 대해 다음과 같이 경고하고 있다

> Fetch should not be used together with setMaxResults() or setFirstResult(), as these operations are based on the result rows which usually contain duplicates for eager collection fetching, hence, the number of rows is not what you would expect.


---

# 🛠️ 해결

```java
return queryFactory
                .selectFrom(user).distinct()
                .leftJoin(user.userRoles, userRole)
                .leftJoin(userRole.role, role)
                .where(applySearchType(condition.getSearchInput(), condition.getSearchType())
                        .and(hasRoles(condition.getRoles()))
                        .and(hasProviders(condition.getProviders()))
                        .and(goeStartDate(condition.getStartDate()))
                        .and(loeEndDate(condition.getEndDate()))
                        .and(isNotDeleted())
                        .and(getCursorCondition(condition.getSortType(), condition.getOrder(), condition.getCursorId()))
                )
                .orderBy(getOrderSpecifier(condition.getSortType(), condition.getOrder()))
                .limit(condition.getSize() + 1)
                .fetch();
```
`user`와 `userRole`은 일대다 관계기 때문에 결과 row가 중복될 수 있다. 따라서 `distinct()`를 사용해 각 사용자가 결과 셋에 한 번만 포함되도록 한다.  
`application.yml`에 `hibernate.default_batch_fetch_size` 적용이 되어 있기 때문에 컬렉션 관계가 설정한 size만큼 `IN` 쿼리로 조회된다. 자세한 내용은 [해당 링크](https://ijnooyah.github.io/spring-data-jpa/role#-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%84%EC%B2%B4-%EC%A1%B0%ED%9A%8C-%ED%8E%98%EC%9D%B4%EC%A7%95-%EC%B5%9C%EC%A0%81%ED%99%94){: target='_blank'}에서 보면 된다. 따라서 따로 `fetchJoin()`을 설정하지 않아도 `1 + N` 문제는 해결이 된다.

---
<p class='ref'>참고</p>
- [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94/dashboard){: target='_blank'}
- [https://velog.io/@sionok/Spring-Querydsl-FetchJoin%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-limit%EC%A0%88%EC%9D%B4-%EB%AC%B4%EC%8B%9C%EB%90%98%EB%8A%94-%EB%AC%B8%EC%A0%9C](https://velog.io/@sionok/Spring-Querydsl-FetchJoin%EC%9D%84-%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4-limit%EC%A0%88%EC%9D%B4-%EB%AC%B4%EC%8B%9C%EB%90%98%EB%8A%94-%EB%AC%B8%EC%A0%9C){: target='_blank'}