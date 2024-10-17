---
title: "[Querydsl] 커서 기반 페이징 구현하기"
excerpt: ""

categories:
  - Querydsl
tags:
  - [Querydsl, 커서 기반 페이징]

permalink: /querydsl/cursor-based-pagination

toc: true
toc_sticky: true

date: 2024-10-17
last_modified_at: 2024-10-17
---

# 📖 커서 기반 페이징 
커서 기반 페이징은 대용량 데이터를 효율적으로 처리하기 위한 페이징 기법이다. 기본 아이디어는 간단하다. 예를 들어, 한 페이지에 10개의 데이터를 보여주고 싶다면, 실제로는 11개의 데이터를 가져온다. 그리고 마지막 데이터(11번째)를 이용해 다음 페이지의 시작점(커서)을 정하고, 다음 페이지가 있는지 없는지도 판단한다. 물론 클라이언트에게는 10개의 데이터만 전송한다. 

필자는 처음에 단순히 사용자의 ID(PK)를 커서 아이디로 사용했다. 그러나 정렬 기능을 추가하면서 문제가 발생했다. 이 문제를 해결하기 위해 좀 더 복잡한 커서 구현 방식을 찾아보았다. ([해당 블로그](https://velog.io/@minsangk/%EC%BB%A4%EC%84%9C-%EA%B8%B0%EB%B0%98-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98-Cursor-based-Pagination-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0){: target='_blank'}에는 커서 기반 페이징이 뭔지 오프셋 기반 페이징에 비해 무엇이 좋은지 잘 설명되어 있으니 참고해보길 바란다.)  

---

# 🔗 정렬과 커서의 관계
정렬을 사용하면 커서는 정렬 기준이 되는 값이 된다. 그러나 여기서 한 가지 문제가 있다. 만약 정렬 기준이 되는 데이터가 중복될 수 있다면 어떻게 될까? 예를 들어, 가입일로 정렬한다고 생각해보자. 같은 날 가입한 사용자가 여러 명 있을 수 있다. 이런 경우에는 정렬 기준 값만으로는 유일성을 보장할 수 없다. 

이 문제를 해결하기 위해 정렬 기준 값과 고유한 값(예: 사용자 ID)을 조합하여 새로운 커서 ID를 만든다. 이렇게 하면 정렬 순서를 유지하면서도 각 항목을 고유하게 식별할 수 있다.

---

# 💻 실제 구현 코드 분석
실습 프로젝트에서 구현한 코드에서는 검색 기능에 정렬 기능을 추가했고, 가입일 또는 이메일로 정렬할 수 있게 만들었다. 
이메일 기반 정렬은 이메일 자체를 고유한 값으로 받고 있으므로 추가 처리 없이 커서 아이디로 사용한다.  
가입일 기반 정렬의 경우 가입일과 사용자 ID(PK)를 조합해 고유한 커서 아이디를 생성한다.

## 🗃️ 리포지토리
```java
 @Override
public List<User> searchUsersWithCursor(AdminUserSearchCondition condition) {
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
}

private BooleanBuilder getCursorCondition(String sortType, String order, String cursorId) {
    if (cursorId == null || cursorId.equals("0")) { // 초기값
        return new BooleanBuilder();
    }

    if (sortType == null) { // sortType이 null일 경우 기본 정렬 (userId)
        return "asc".equalsIgnoreCase(order) ?
                new BooleanBuilder(user.id.goe(Long.parseLong(cursorId))) :
                new BooleanBuilder(user.id.loe(Long.parseLong(cursorId)));
    }

    StringExpression stringExpression;

    // sortType이 CREATED_AT일때
    if (SortType.CREATED_AT.name().equalsIgnoreCase(sortType)) {
        StringTemplate stringTemplate = Expressions.stringTemplate(
                "cast(DATE_FORMAT({0}, {1}) as char)",
                user.createdAt,
                ConstantImpl.create("%y%m%d%H%i%s")
        );
        stringExpression = StringExpressions.lpad(stringTemplate, 20, '0')
                        .concat(StringExpressions.lpad(user.id.stringValue(), 10, '0'));
        return "asc".equalsIgnoreCase(order) ?
                new BooleanBuilder(stringExpression.goe(cursorId)) :
                new BooleanBuilder(stringExpression.loe(cursorId));
    }

    // sortType이 EMAIL일때
    if (SortType.EMAIL.name().equalsIgnoreCase(sortType)) {
        return "asc".equalsIgnoreCase(order) ?
                new BooleanBuilder(user.email.goe(cursorId)) :
                new BooleanBuilder(user.email.loe(cursorId));
    }

    return new BooleanBuilder();
}
```

`getCursorCondition` 메서드의 주요 특징
- 초기 페이지 처리: 커서 아이디가 없거나 0이면 조건 없이 첫 페이지를 반환한다.
- 정렬 타입별 처리:
  - 정렬 타입이 없으면 기본적으로 사용자 ID로 정렬한다.
- 정렬 방향 고려: 오름차순(ASC)과 내림차순(DESC)에 따라 적절한 비교 연산자(>=, <=)를 사용한다.

`getCursorCondition` 메서드에서 주목해야 할 코드를 살펴보자:

```java
// sortType이 CREATED_AT일때
if (SortType.CREATED_AT.name().equalsIgnoreCase(sortType)) {
    StringTemplate stringTemplate = Expressions.stringTemplate(
            "cast(DATE_FORMAT({0}, {1}) as char)",
            user.createdAt,
            ConstantImpl.create("%y%m%d%H%i%s")
    );
    stringExpression = StringExpressions.lpad(stringTemplate, 20, '0')
                    .concat(StringExpressions.lpad(user.id.stringValue(), 10, '0'));
    return "asc".equalsIgnoreCase(order) ?
            new BooleanBuilder(stringExpression.goe(cursorId)) :
            new BooleanBuilder(stringExpression.loe(cursorId));
}
```
가입일을 'yyMMddHHmmss' 형식으로 변환하고, 20자리로 왼쪽을 0으로 채운 후, 사용자 ID를 10자리로 왼쪽을 0으로 채운 값과 연결한다. 이렇게 하면 가입일과 사용자 ID를 조합한 유일한 커서 ID를 생성할 수 있다.

예를들어 오름차순일 때 커서 아이디가 000000002409282020510000000034일 때 실행되는 쿼리를 SQL 문으로 보면 다음과 같다:
```sql
SELECT DISTINCT 
    u1_0.user_id,
    u1_0.created_at,
    u1_0.deleted,
    u1_0.deleted_at,
    u1_0.email,
    u1_0.nickname,
    u1_0.password,
    u1_0.file_id,
    u1_0.provider,
    u1_0.updated_at
FROM 
    users u1_0
LEFT JOIN 
    user_role ur1_0 ON u1_0.user_id = ur1_0.user_id
WHERE 
    (
        LOWER(u1_0.email) LIKE '%gmail%' ESCAPE '!' 
        OR LOWER(u1_0.nickname) LIKE '%gmail%' ESCAPE '!'
    )
    AND EXISTS (
        SELECT 1 
        FROM 
            user_role ur2_0
        JOIN 
            role r2_0 ON r2_0.role_id = ur2_0.role_id
        WHERE 
            r2_0.role_name IN ('ROLE_USER', 'ROLE_ADMIN', 'ROLE_MANAGER') 
            AND u1_0.user_id = ur2_0.user_id
    )
    AND u1_0.provider IN ('GOOGLE', 'NAVER', 'LOCAL')
    AND u1_0.created_at >= '2024-09-01T00:00:00.000+0900'
    AND u1_0.created_at <= '2024-10-01T00:00:00.000+0900'
    AND (
        u1_0.deleted = FALSE 
        AND u1_0.deleted_at IS NULL
    )
    AND CONCAT(
        LPAD(CAST(DATE_FORMAT(u1_0.created_at, '%y%m%d%H%i%s') AS CHAR), 20, '0'),
        LPAD(CAST(u1_0.user_id AS CHAR), 10, '0')
    ) >= '000000002409282020510000000034'
ORDER BY 
    u1_0.created_at
LIMIT 11;
```

---
{: .style1}

## ⚙️ 서비스
서비스 레이어에서 리포지토리로부터 받은 결과를 처리하고, 다음 페이지 존재 여부와 다음 커서 ID를 결정한다.

```java
@Transactional(readOnly = true)
public AdminUserListResponse searchUsersWithCursor(AdminUserSearchCondition condition) {
    List<User> users = userRepository.searchUsersWithCursor(condition);

    boolean hasNext = false;
    String nextCursorId = null;

    if (users.size() > condition.getSize()) {
        hasNext = true;
        Long userId = users.getLast().getId();

        nextCursorId = switch (SortType.valueOf(condition.getSortType())) {
            case CREATED_AT -> generateCreatedAtCursorId(users.getLast().getCreatedAt(), userId);
            case EMAIL -> users.getLast().getEmail();
        };
    }

    List<AdminUserResponse> adminUserResponse = users.stream()
            .limit(condition.getSize())
            .map(this::convertToAdminUserResponse)
            .collect(Collectors.toList());


    return AdminUserListResponse.builder()
            .users(adminUserResponse)
            .hasNext(hasNext)
            .nextCursorId(nextCursorId) // 다음 커서 ID 설정
            .build();
}

private String generateCreatedAtCursorId(LocalDateTime createdAt, Long userId){
    // 1. LocalDateTime을 DATE_FORMAT와 동일한 포맷으로 변환 (yyMMddHHmmss)
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyMMddHHmmss");
    String formattedCreatedAt = createdAt.format(formatter);

    // 2. 포맷된 createdAt을 20자리로 왼쪽을 '0'으로 채움
    String customCursorCreatedAt = StringUtils.leftPad(formattedCreatedAt, 20, '0');

    // 3. userId를 문자열로 변환하고 10자리로 왼쪽을 '0'으로 채움
    String customCursorId = StringUtils.leftPad(userId.toString(), 10, '0');

    // 4. 두 값을 연결하여 반환
    return customCursorCreatedAt + customCursorId;
}
```

설명:
- 리포지토리에서는 요청한 크기보다 하나 더 많은 항목을 조회한다.(10개 요청시 11개 조회)
- 결과 크기가 요청 크기보다 크면 다음 페이지가 있다고 판단한다.
- 다음 페이지가 있을 경우, 마지막 항목을 이용해 다음 커서 ID를 생성한다. (`generateCreatedAtCursorId` 메서드)
- 실제 응답에는 요청한 크기만큼의 항목만 포함 시킨다. (11개의 데이터에서 마지막 항목 제외)

---

<p class='ref'>참고</p>
- [https://velog.io/@minsangk/%EC%BB%A4%EC%84%9C-%EA%B8%B0%EB%B0%98-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98-Cursor-based-Pagination-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0](https://velog.io/@minsangk/%EC%BB%A4%EC%84%9C-%EA%B8%B0%EB%B0%98-%ED%8E%98%EC%9D%B4%EC%A7%80%EB%84%A4%EC%9D%B4%EC%85%98-Cursor-based-Pagination-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0){: target='_blank'}