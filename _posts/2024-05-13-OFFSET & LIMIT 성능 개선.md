---
categories: [BackEnd, Paging, OFFSET & LIMIT]
tags: [BackEnd, OFFSET & LIMIT, Paging]
---

## OFFSET & LIMIT의 문제점
- 페이지를 조회할 때마다 `제일 첫번째 데이터`부터 `limit까지의 데이터`를 검색하게 된다.
- 그 후에 `OFFSET`으로 설정된 값만 가지고 `나머지는 버린다.`

![img.png](../assets/img/postimg/2024-05-13/페이지%20이동.png)

`0 ~ 2까지의 데이터`를 조회할 때 `0부터`시작  
`9990 ~ 10000까지의 데이터`를 조회할 때도 `0부터`시작 한다.


- 이로 인해 데이터의 양이 많아 지면 많아 질수록 쿼리의 속도가 느려지게 된다.
- 사실상 몇천 몇만 건이 되는 데이터를 가지고 있지 않으면 유의미한 수치가 아닐 수 있지만, 
  `문제가 발생하고 조치하는 것보다 미연에 방지하는 것`이 더 좋을 것 같아 문제점을 해결해 보려고 한다.

<br>

---

<br>

### 문제가 발생하는지 테스트

- 실제로 문제가 발생하는지 환경을 만들어 테스트 해보도록 했다.
- 조건 : `한 페이지당 4건`의 데이터를 조회하면 `약 2만건`의 데이터를 가진 `테이블을 조회`

<br>

`1페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/2만개%20이상일%20때%201페이지%20요청속도.PNG)

`약 85ms`

<br>

---

<br>

`400페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/2만개%20이상일%20때%20400%20페이지%20요청속도.PNG)

`약 89ms`

<br>

---

`500페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/2만개%20이상일%20때%20500%20페이지%20요청속도.PNG)

`약 128ms`


<br>

**<span style="color: #4682B4;">약간의 오차 범위는 존재 하지만 페이지를 넘기면 넘길 수록 조회 속도가 느려 지는걸 확인할 수 있었다.</span>**

현재는 사람이 느낄 수 있을 정도로 느리지는 않지만 `데이터가 늘어 나면 확실히 느려질 거`라는 확신이 생겼다.

<br>

### 해결 방안

<br>

해결 방안은 크게 `2가지`가 있었다.  

<br>

1 - NoOffset 방식
- NoOffset 방식은 `SNS인 유튜브나 인스타그램`에서 많이 사용하는 스크롤 UI에서 많이 사용된다.
- 기존 페이징 방식은 `페이지 번호(offset)`와 `페이지 사이즈(limit)`를 기반 이지만   
  조회 시작 부분을 `인덱스`로 찾고 `매번 첫 페이지를 읽도록` 하는 방식 

<br>

2 - 커버링 인덱스
- 필요한 모든 데이터가 `인덱스`에서만 추출할 수 있도록 하는 방식
- `PK로 설정된 컬럼(clustered index)`은 이미 정렬이 되어 있기 때문에 `index 생성 필요 없음`

<br>

해결 방안으로 <span style="color: #4682B4;">**커버링 인덱스 방식**</span>으로 선택했다.  
이유는 `NoOffset 방식`은 `총 페이지 수`와 `현재 페이지 번호`를 사용하지 않고 `인덱스의 범위`를 지정해 줘야 하기 때문에 
구조를 완전히 수정해야 한다는 번거로움이 존재했다.  
  
반면 `커버링 인덱스`는 현재 `페이지 번호(offset)`와 `페이지 사이즈(limit)`를 그대로 사용할 수 있기에 선택했다.







<br>

---

## Querydsl 작성

```java
List<RoomPost> roomPostList = jpaQueryFactory
        .selectFrom(roomPost)
        .join(roomPost.member, member)
        .where(containsSearch(searchOption, searchContent))
        .limit(pageable.getPageSize())
        .offset(pageable.getOffset())
        .orderBy(roomPost.createAt.desc())
        .fetch();
```

`limit`과 `offset`을 사용해서 `member`테이블만 조인해서 사용하고 있다.    


<br>

```mysql
SELECT *
FROM room_post as rp
JOIN (
	SELECT id
	FROM room_post
	WHERE 조건문
	ORDER BY id DESC
	OFFSET 페이지번호
	LIMIT 페이지사이즈
) as rpids
ON rpids.id = rp.id
```
`커버링 인덱스`을 구성하기 위한 쿼리문  

보통 `SELECT`절은 너무 많은 칼럼을 인덱스로 포함시킬 수 있어서 `JOIN`에서 `커버링 인덱스`를 작성하는데 `room_post`의 `id`를 인덱스로 활용했다.  

<br>

하지만 `JPQL`은 `from`절에 서브쿼리를 작성할 수 없기 때문에 `JOIN`에 `커버링 인덱스`를 작성할 수 없었다.  
그래서 `커버링 인덱스`용도의 쿼리를 따로 작성하고 필요한 조건들도 미리 처리해 준다.  
그 후, `WHERE`에서 `in`을 사용해 `커버링 인덱스`를 가져와 조회하는 방법으로 처리 했다.  

<br>

```java
List<Long> roomPostIds = jpaQueryFactory
                .select(roomPost.id)
                .from(roomPost)
                .where(containsSearch(searchOption, searchContent))
                .orderBy(roomPost.id.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        List<RoomPost> roomPostList = jpaQueryFactory
                .selectFrom(roomPost)
                .join(roomPost.member, member)
                .where(roomPost.id.in(roomPostIds))
                .fetch();
```

<br>

---

<br>

## 개선

<br>

개선 후에 같은 조건으로 테스트 해보았다.
- 조건 : `한 페이지당 4건`의 데이터를 조회하면 `약 2만건`의 데이터를 가진 `테이블을 조회`

<br>

`1페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/개선%201페이지.PNG)

`약 19ms`

<br>

---

<br>

`400페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/개선%20400페이지.PNG)

`약 19ms`

<br>

---

<br>

`500페이지`를 조회.

![img.png](../assets/img/postimg/2024-05-13/개선%20500페이지.PNG)

`약 18ms`

<br>

거의 균등하게 조회되는 것을 확인할 수 있었다.   
오차 범위도 `3ms ~ 7ms`정도 밖에 나지 않는 것 같다.  

<br>

`500페이지`를 조회하는 기준 `약 128ms`에서 `약 18ms`까지 조회 시간을 줄일 수 있었다.  
적은 작업량으로 속도를 `약 85%`개선 할 수 있었다.  












