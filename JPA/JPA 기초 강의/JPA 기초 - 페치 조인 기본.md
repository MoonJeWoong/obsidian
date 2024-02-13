
페치 조인은 JPA에서 성능 개선에 가장 많이 사용되는 기능이다.

한 엔티티를 조회할 때 연관된 다른 엔티티나 컬렉션을 한 번의 쿼리로 같이 조회해오는 것.

페치 조인은 JPQL을 사용하면서 지연로딩으로 인한 N+1 문제가 발생할 때 객체 그래프를 한 번의 쿼리로 조회해오는 것으로 해결한다.

ManytoOne 연관관계를 맺는 member와 team의 경우에서 member를 조회하기 위해 team을 페치 조인하게 되면 member 개수 만큼 row가 조회된다.

하지만 OneToMany 연관관계인 team과 member의 경우에서 team을 조회하기 위해 페치 조인하게 되면 같은 팀 소속 member가 존재하는 경우 같은 team 엔티티가 중복되어 조회된다.

여기에서 조회 결과 엔티티는 하나로 동일한데 결과 컬렉션에 동일한 엔티티를 가리키는 여러 개의 공간이 할당된 것이다. 

```java

@Entity
public class Team {
	@OneToMany
	private List<Member> members = new ArrayList<>();
}

@Entity
public class Member {
	@ManyToOne(fetch = FetchType.LAZY)
	private Team team;
}

@Repository
public class TeamRepository {
	
	private final EntityManager em;

	public Team findByName(String name) {
		List<Team> teams = em.createQuery("select t from Team t join fetch t.members where t.name = :name", Team.class)
		.setParameter("name", name)
		.getResultList();
		
	}
}
```

teamRepository 코드에서 조회 jpql을 수행하면 아래와 같이 팀A라는 조회 결과가 중복되어 반환된다. 영속성 컨텍스트에서는 팀A라는 엔티티가 하나만 존재하지만 조회 결과 컬렉션에는 동일한 팀A라는 엔티티를 여러번 가리키게 된다는 것이다.

![](https://i.imgur.com/JSj75wQ.png)


이러한 중복을 제거하기 위해 distinct 키워드를 사용할 수 있다.

SQL 수준에서 distinct는 완전히 값이 동일한 row만 제거해 주지만, 어플리케이션 단에서 JPA가 동일한 엔티티라고 판단하는 경우도 삭제시켜 준다.

일반 조인 쿼리와 페치 조인 쿼리의 차이

일반 조인 쿼리가 실행되는 경우 select 문에서 명시한 엔티티만 조회해오지만 페치 조인으로 실행하는 경우 연관관계에 있는 엔티티를 select 문에서 명시하지 않더라도 전부 조회해서 가져온다. (즉시 로딩에서 나가는 쿼리의 작용과 동일하다.)

추가사항

하이버네이트6 부터는 DISTINCT 명령어를 사용하지 않아도 애플리케이션 단에서 중복 제거가 자동으로 적용된다고 한다.