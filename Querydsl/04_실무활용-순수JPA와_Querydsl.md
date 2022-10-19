### 실전! Querydsl

#### Reference) 
	* 실전! Querydsl (인프런)

#### 작성 코드
- https://github.com/hesongg/Querydsl-Practice

<br>

### 실무 활용 - 순수 JPA와 Querydsl

<br>

#### 순수 JPA 리포지토리와 Querydsl

- 순수 JPA 리포지토리로 작성한 코드 일부
	```java
	private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }

    public void save(Member member) {
        em.persist(member);
    }

    public Optional<Member> findById(Long id) {
        Member findMember = em.find(Member.class, id);
        return Optional.ofNullable(findMember);
    }

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where :username = m.username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }
	```

<br>

- 순수 JPA 리포지토리 - Querydsl 추가
	```java
	public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member)
                .fetch();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
	```

<br>

- ```JPAQueryFactory``` 스프링 빈 등록
	
	- 다음과 같이 ```JPAQueryFactory``` 를 스프링 빈으로 등록해서 주입받아 사용해도 된다.
		```java
		@Bean
		JPAQueryFactory jpaQueryFactory(EntityManager em) {
			return new JPAQueryFactory(em);
		}
		```
	
	- 참고
		- 동시성 문제는 걱정하지 않아도 된다. 
		
		- 왜냐하면 여기서 스프링이 주입해주는 엔티티 매니저는 실제 동작 시점에 진짜 엔티티 매니저를 찾아주는 
			프록시용 가짜 엔티티 매니저이다. 
			
		- 이 가짜 엔티티 매니저는 실제 사용 시점에 트랜잭션 단위로 실제 엔티티 매니저(영속성 컨텍스트)를 할당해준다.
	
<br>

#### 동적 쿼리와 성능 최적화 조회 - Builder 사용

- ```MemberTeamDto``` - 조회 최적화용 DTO 추가
	
	```java
	@Data
	public class MemberTeamDto {

		private Long memberId;
		private String username;
		private int age;
		private Long teamId;
		private String teamName;

		@QueryProjection
		public MemberTeamDto(Long memberId, String username, int age, Long teamId, String teamName) {
			this.memberId = memberId;
			this.username = username;
			this.age = age;
			this.teamId = teamId;
			this.teamName = teamName;
		}
	}
	```
	
	- ```@QueryProjection``` 을 추가했다. 
		- ```QMemberTeamDto``` 를 생성하기 위해 ```./gradlew compileQuerydsl``` 을 한번 실행하자.
	
	- 참고
		- ```@QueryProjection``` 을 사용하면 해당 DTO가 Querydsl을 의존하게 된다. 
		- 이런 의존이 싫으면, 해당 에노테이션을 제거하고, 
			```Projection.bean(), fields(), constructor()``` 을 사용하면 된다.

<br>

- 회원 검색 조건
	```java
	@Data
	public class MemberSearchCondition {
		//회원명, 팀명, 나이(ageGoe, ageLoe)
		
		private String username;
		private String teamName;
		private Integer ageGoe;
		private Integer ageLoe;
	}
	```

<br>

- 동적쿼리 - ```Builder``` 사용

	- Builder를 사용한 예제
		```java
		public List<MemberTeamDto> searchByBuilder(MemberSearchCondition condition) {

			BooleanBuilder builder = new BooleanBuilder();
			if (hasText(condition.getUsername())) {
				builder.and(member.username.eq(condition.getUsername()));
			}
			if (hasText(condition.getTeamName())) {
				builder.and(team.name.eq(condition.getTeamName()));
			}
			if (condition.getAgeGoe() != null) {
				builder.and(member.age.goe(condition.getAgeGoe()));
			}
			if (condition.getAgeLoe() != null) {
				builder.and(member.age.goe(condition.getAgeLoe()));
			}

			return queryFactory
					.select(new QMemberTeamDto(
							member.id,
							member.username,
							member.age,
							team.id,
							team.name
					))
					.from(member)
					.leftJoin(member.team, team)
					.where(builder)
					.fetch();
		}
		```

<br>

#### 동적 쿼리와 성능 최적화 조회 - Where절 파라미터 사용

- Where절에 파라미터를 사용한 예제
	```java
	public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name
                ))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe != null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe != null ? member.age.loe(ageLoe) : null;
    }
	```

<br>

- 테스트 수행한 코드
	```java
	@Test
    public void searchTest2() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");
        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);
        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);

        MemberSearchCondition condition = new MemberSearchCondition();
        //        condition.setAgeGoe(35);
        //        condition.setAgeLoe(40);
        condition.setUsername("member3");
        condition.setTeamName("teamB");

        List<MemberTeamDto> result = memberJpaRepository.search(condition);

        assertThat(result).extracting("username").containsExactly("member3");
    }
	```

<br>
	
- 참고 : where 절에 파라미터 방식을 사용하면 조건 재사용 가능

<br>

#### 조회 API 컨트롤러 개발

- 편리한 데이터 확인을 위해 샘플 데이터를 추가
	- 샘플 데이터 추가가 테스트 케이스 실행에 영향을 주지 않도록 다음과 같이 프로파일을 설정

<br>

- 프로파일 설정
	- test 와 별도의 프로파일 설정 추가
		```yml
		spring:
			profiles:
				active: local
		...
		```
	
	- 이렇게 분리하면 main 소스코드와 테스트 소스 코드 실행시 프로파일을 분리 가능

<br>

- ```local``` 프로파일에서 동작하는 샘플 데이터 추가 코드 작성 
	- 커밋한 ```InitMember``` 코드 참고

- 간단한 ```Member``` 리스트 조회 컨트롤러 작성
