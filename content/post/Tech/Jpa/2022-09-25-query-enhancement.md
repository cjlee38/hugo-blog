---
categories: jpa
date: "2022-09-25T00:00:00Z"
tags: ['jpa', query']
title: '# 모락 쿼리 성능 개선기'
---

# 0. 서론

우아한테크코스의 레벨4, 5차 데모에서 주어진 요구사항 중 하나는 아래와 같습니다.

- 서비스에서 사용하는 쿼리를 정리하고, 각 쿼리에서 사용하는 인덱스 설정
- 서비스에서 사용하는 모든 조회 쿼리와 테이블에 설정한 인덱스 공유
- 인덱스를 설정할 수 없는 쿼리가 있는 경우, 인덱스를 설정할 수 없는 이유 공유

레벨 3 기간 동안 쿼리가 어떻게 나가는지에 대해서 전혀 신경을 쓰지 않았었습니다. 20 만 건의 더미 데이터를 넣어놓고 쿼리 개수와 시간을 측정해보았을 때의 결과는 처참했죠. 아래는 저희가 측정한 API 성능표입니다.

| |Mean|Min|Max|
|:--:|:--:|:--:|:--:|
|요청 처리 시간|307.872|16.496|3234.609|
|쿼리 시간|236.812|1.437|3133.169
|쿼리 개수|43.6|2|1002|

평균 처리 시간이 짧은지 느린지는 확언할 수 없지만, 최대 처리 시간이 약 3초 남짓, 쿼리 개수가 1002개가 나가는 모습은 분명 문제가 있다고 이야기할 수 있습니다. 각각의 API 에서 요청이 어떻게 처리되는지 확인한 뒤, 하나씩 해결해본 경험을 말씀드리겠습니다.

# 1. ID 기반 조회
저희 서비스에서 각 entity 는 서로 직접 참조를 맺고 있었습니다. 가령 `Poll` 이라는 entity 안에 `Team` 이라는 필드를 직접 가지고 있는 형태였죠. 그리고 repository 에서는 다음과 같은 메소드를 쓰고 있었습니다. 

```java
// PollRepository#findAllByTeamId
List<Poll> findAllByTeamId(Long teamId);
``` 

위 메소드는 얼핏 보면 아무런 문제가 없어보입니다. ID 값을 기반으로 조회해오겠다는 뜻이니까요. 하지만 실제로 나가는 쿼리는 다음과 같았습니다.

```sql
select
	poll0_.id as id1_3_,
	poll0_.created_at as created_2_3_,
	poll0_.updated_at as updated_3_3_,
	...
from
	poll poll0_ 
left outer join
	team team1_ 
		on poll0_.team_id=team1_.id 
where
	team1_.id=?
```

분명 `poll` 이라는 테이블안에는 `team_id` 라는 외래키가 존재함에도 불구하고 `left outer join` 이 불필요하게 나가는 모습을 확인할 수 있습니다. 종종 `JPQL`과 `SQL`을 헷갈리고 이런 식으로 작성하는 경우가 많은데요. `poll` 이라는 entity가 갖고있는 객체는 `team_id`가 아닌 `team`이고, 따라서 team이 갖고 있는 다시 갖고 있는 id 를 기반으로 조회하고자 위와 같은 쿼리가 나가게 되는 것입니다. 

개선 방법은 간단하게, entity를 기반으로 조회하도록 변경하는 것입니다. 

```sql
select
	poll0_.id as id1_3_,
	poll0_.created_at as created_2_3_,
	poll0_.updated_at as updated_3_3_,
	...
from
	poll poll0_ 
where
	poll0_.team_id=?
```

따라서 불필요한 join 쿼리 없이 곧바로 where 절로 접근할 수 있게 되었습니다.

# 2. 단건 처리 (1)

사용하던 메소드 시그니쳐는 아래와 같습니다.

```sql
AvailableTimeRepository#deleteAllByAppointment
void deleteAllByAppointment(Appointment appointment)
``` 

이에 대응하는 쿼리는 `delete from appointment_available_time where appointment_id = ?` 으로, 즉 appointment를 where 절에 걸어서 delete 쿼리가 나가길 기대했지만, 실제 실행 결과는 아래와 같았습니다.

![](/assets/images/2022-09-25-query-enhancement/2022-10-05-11-05-29.png)

위와 같이 `appointment_id` 가 아닌 `id` 를 기반으로 `available_time`을 삭제하고 있었습니다. 그 이유는 구현체인 `SimpleJpaRepository` 를 보면 유추할 수 있습니다.

![](/assets/images/2022-09-25-query-enhancement/2022-10-05-11-34-19.png)

deleteAll 메소드 자체는 단순히 entity의 목록을 반복하면서 삭제하기를 반복하고만 있었습니다. 이를 해결하기 위해서 직접 JPQL을 작성해주었습니다.

```java
@Modifying
@Query("DELETE FROM AvailableTime at WHERE at.appointment = :appointment")
void deleteAllByAppointment(@Param("appointment") Appointment appointment);
```

직접 작성한 `JPQL` 의 결과로 날아가는 쿼리는 아래와 같습니다. 

```sql
delete 
from
	appointment_available_time 
where
	appointment_id=?
```

> `deleteAll` 메소드 바로 밑에 `deleteAllInbatch` 라는 메소드가 있지만, `deleteAllInBatchByAppointment`와 같이 활용할 수 없는 것으로 파악했습니다.
> ![](/assets/images/2022-09-25-query-enhancement/2022-10-05-11-40-51.png)
> [공식 문서에서도 `@Query` 를 이용해서 처리하고 있다는 것](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.modifying-queries.derived-delete)을 확인할 수 있었습니다. 혹시 `InBatch` clause 를 이용할 수 있는 방법이 있다면 댓글 부탁드립니다.

# 3. 단건 처리 (2)

여러 이유로 영속성 컨텍스트에 엔티티를 담을수 없어, 직접  `update` 메소드를 repository 를 통해 직접 처리하고 있는 경우가 있었습니다. 대략 다음과 같이 구성되어 있었습니다.

```java
// PollRepository#closedById 시그니쳐
@Query("update Appointment a set a.status = 'CLOSED' where a.id = :id")
void closeById(@Param("id") Long id);

// 사용할 때
for (Poll poll: polls) {
	pollRepository.closed(poll.getId()))
}
```

앞서 2번과 비슷하게도, 이 부분 또한 반복문을 통해 처리되기 때문에 단건으로 처리됩니다. 어차피 업데이트할 엔티티는 이미 정해져있기 떄문에 굳이 번거롭게 매번 쿼리를 날려 불필요한 딜레이 시간을 가질 필요가 없습니다. 따라서 다음과 같이 수정하였습니다.

```java
@Modifying
@Query("update Appointment a set a.status = 'CLOSED' where a in :appointments")
void closeAll(@Param("appointments") Iterable<Appointment> appointments);
```

`JPQL` 에서는 위와 같이 `in` 절도 처리할 수 있습니다.[^1]


# 4. N+1 문제

`N+1 문제`는 JPA를 사용하면서 가장 흔하게 발생하고 또 신경써줘야 하는 문제입니다. 모든 N+1 문제를 해결하면서 동시에 ORM의 장점을 취하는 silver bullet은 존재하지 않고, 본인이 사용하는 로직이 N+1 문제가 어떻게 발생하느냐에 따라 적절한 해결책을 선택해야 합니다. N+1 문제를 해결하기 위한 근본적인 해결책은 entity 간의 직접적인 참조를 하지 않는 것이지만, `ORM` 이라는 기술이 제공해주는 다양한 장점을 불가피하게 버릴 수밖에 없습니다. 

모락의 도메인에서 `Team` 은 여러 `Member` 를 가질 수 있고, `Member` 또한 여러 `Team` 을 가질 수 있습니다. 이러한 다대다 관계를 풀어내기 위해 `TeamMember`라는 중간 엔티티(테이블)를 두었습니다. 그리고, `Member`가 속해있는 `Team`의 목록을 조회하기 위해서는 아래와 같은 로직을 사용했습니다.

```java
// TeamMemberRepository#findAllByMember
List<TeamMember> findAllByMember(Member member);

// 사용할 때
List<TeamMember> teamMembers = teamMemberRepository.findAllByMember(member);
List<Team> teams = teamMembers.stream()
		.map(teamMember -> teamMember.getTeam())
		.collect(Collectors.toList());
```

`TeamMember`를 조회하는 것 까지는 좋았으나, 여러 `TeamMember`에 대해서 각각의 `team`을 구해온다면 N+1 문제가 발생합니다. 가령, 1명의 멤버가 10개의 팀에 속해있다고 가정해봅시다. 그렇다면 전체 쿼리는 다음과 같이 구성됩니다.

1. Member 조회 (1번)
   `select * from member where id = ?`
2. Member가 속해 있는 TeamMember 목록을 조회 (1번)
   `select * from team_member where member_id = ?`
3. TeamMember 목록에서 각각의 Team을 조회
   `select * from team where id = ?` * 10

따라서 총 12 번의 쿼리가 날아갑니다. 만약 소속된 Team이 1000개 였다면 총 1002번의 쿼리가 날아갔겠죠. 다행히 아직까지는 저희 도메인 정책상 페이지네이션과 같은 로직은 존재하지 않기 때문에, `fetch join`을 이용하여 비교적 간단하게 해결할 수 있었습니다.

```java
@Query("SELECT tm FROM TeamMember tm JOIN FETCH tm.team WHERE tm.member = :member")
List<TeamMember> findAllByMember(@Param("member") Member member);
```

위와 같은 `JPQL` 을 작성한다면 `teamMember` entity를 조회하는 과정에서 `team` entity를 join해서 가져오게 됩니다. 따라서 1000 개의 `team`이 있다 하더라도 쿼리 개수 자체는 3번에서 끝나게 됩니다.

> `@EntityGraph` 를 사용해도 `N+1 문제`는 해결할 수 있지만, 이에 대한 쿼리를 살펴보면 `left outer join` 을 사용합니다.
> {{< figure src="/assets/images/2022-09-25-query-enhancement/2022-10-07-09-21-45.png" width="400px" align="center" >}} 
> 하지만 직접 `fetch join`을 사용하게 되면 `inner join` 을 사용합니다.
> {{< figure src="/assets/images/2022-09-25-query-enhancement/2022-10-07-09-23-12.png" width="400px" align="center" >}} 


# 5. 최종 결과
개선 과정을 거친 뒤, 최종 성능 측정 결과 표는 아래와 같습니다.


| |Mean|Min|Max|
|:--:|:--:|:--:|:--:|
|요청 처리 시간|37.478|9.44|219.352|
|쿼리 시간|1.698|0.307|7.87|
|쿼리 개수|5.6|2|13|


평균을 기준으로, 각각 약 8, 140, 7 배 가량 개선하였습니다.


[^1]: [DBA와 개발자가 모두 행복해지는 Hibernate의 in_clause_parameter_padding 옵션](https://meetup.toast.com/posts/211)
