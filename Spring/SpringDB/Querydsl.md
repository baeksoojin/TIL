# Querydsl이란 무엇일까?

- Querydsl이란 쿼리 단순화 Framework이다,

최신 자바 백엔드에서 동적쿼리를 다루기 위해서 querydsl을 작성한다.

- 장점이 무엇일까?

query를 문자가 아닌 java code로 작성해서 문법 오류를 컴파일 시점에 잡아준다.(run time error check)

동적쿼리를 작성하기 쉬워진다.

---

## Querydsl을 어떻게 사용해야하나?

### **프로젝트 환경 설정**부터 알아보자.

library를 추가

plugin 추가

build와 관련된 세팅

```java
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
	jpa = true
	querydslSourcesDir = querydslDir
}
sourceSets {
	main.java.srcDir querydslDir
}
compileQuerydsl {
	options.annotationProcessorPath = configurations.querydsl
}
configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
	querydsl.extendsFrom compileClasspath
}
```

자동으로, entity를 보고 Qentity file을 생성해준다.

querydsl이 generate해준 Q 파일은 git file로 관리해주면 안 된다. ⇒ `gitignore`

다행히, build는 ignore해서 쓰기 때문에 build file안의 Q file을 **따로 ignore안 해줘도된다.**

위의 설정을 통해서 세팅하면 build folder 안에 generated안에 querydsl안에 Q file이 생성된다.

### 간단한 테스트를 해보자

- entity

```java
@Entity
@Getter @Setter
public class Hello {

    @Id @GeneratedValue
    private Long id;
}
```

compileQuerydsl을 동작 → build파일 내에 Q 파일이 생성된다.

이때 생성된 Q 파일을 사용하여 querydsl을 작성하면 된다.

- QHello

```java
@Generated("com.querydsl.codegen.DefaultEntitySerializer")
public class QHello extends EntityPathBase<Hello> {

    private static final long serialVersionUID = -326097488L;

    public static final QHello hello = new QHello("hello");

    public final NumberPath<Long> id = createNumber("id", Long.class);

    public QHello(String variable) {
        super(Hello.class, forVariable(variable));
    }

    public QHello(Path<? extends Hello> path) {
        super(path.getType(), path.getMetadata());
    }

    public QHello(PathMetadata metadata) {
        super(Hello.class, metadata);
    }

}
```

- testcode

```java
@SpringBootTest
@Transactional
class QuerydslApplicationTests {

	@Autowired
	EntityManager em;

	@Test
	void contextLoads() {
		Hello hello = new Hello();
		em.persist(hello);

		JPAQueryFactory query = new JPAQueryFactory(em);
		QHello qHello = new QHello("h");

		Hello result = query
				.selectFrom(qHello)
				.fetchOne();

		Assertions.assertThat(result).isEqualTo(hello);
		Assertions.assertThat(result.getId()).isEqualTo(hello.getId());
	}

}
```

→ 정상동작함을 체크하면 됨

---

### 그렇다면, JPQL과 Querydsl의 차이점이 무엇인지 알아보자!

- TEST 생성
1. JPQL

```java
@Test
public void startJPQL(){
    //member1 find
    String qlString = "select m from Member m "
            +"where m.username = :username";

    Member findMember = em.createQuery(qlString, Member.class)
            .setParameter("username","member1")
            .getSingleResult();

    Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
}
```

- runtime error로 체크

1. Querydsl

```java
@Test
public void startQuerydsl(){
    //member1 find
    JPAQueryFactory queryFactory = new JPAQueryFactory(em);
    QMember m = new QMember("m");

    Member findMember = queryFactory
            .select(m)
            .from(m)
            .where(m.username.eq("member1"))
            .fetchOne();

    Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");

}
```

- 자동으로 paramether binding방식을 사용

![image](https://user-images.githubusercontent.com/74058047/231783119-d95d69d8-58b2-422c-af9b-ceb6d15cbd04.png)


- compile 시점에서 오류체크

-----


## Querydsl 기본문법에 대해서 알아보자

### 생성방법

1. 별칭

```java
QMember m = new QMember("m");

 Member findMember = queryFactory
            .select(m)
            .from(m)
            .where(m.username.eq("member1"))
            .fetchOne();
```

1. instance에 접근해서 사용

```java

Member findMember = queryFactory
      .select( QMember.member)
      .from( QMember.member)
      .where( QMember.member.username.eq("member1"))
      .fetchOne();
```

⇒ **static import를 진행[권장방식]**

```java
import static tutorial.querydsl.entity.QMember.member;
```

```java
Member findMember = queryFactory
                .select( member)
                .from( member)
                .where( member.username.eq("member1"))
                .fetchOne();
```

### 검색조건 쿼리 사용방법

```java
@Test
public void search(){
    Member findMember = queryFactory
            .selectFrom(member) //select+from -> selectFrom
            .where(member.username.eq("member1")
                    .and(member.age.eq(10)))
            .fetchOne();

    Assertions.assertThat(findMember.getUsername()).isEqualTo("member1");
}
```

- 결과조회 방법

1. fetch() : 리스트조회 → 없다면 빈 리스트를 조회한다.
2. fetchOne() → 단건조회 → 없다면 null이지만 둘 이상이면 NonUniqueException
3. fetchFirst() : limit(1).fetchOne()
4. fetchResults() : 페이징 정보를 포함하고 total count 쿼리를 추가 실행한다.
5. fetchCount() : count쿼리로 변경해서 count값을 조회한다.

- 순서 사용 방법

```java
List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(100))
                .orderBy(member.age.desc(), member.username.asc().nullsLast())
                .fetch();
```

회원이름을 내림차순 정렬후에 이름으로 오름차순 정렬하는데 이름이 없다면 마지막으로 출력

- paging기능

```java
@Test
public void paging1(){
    List<Member> result = queryFactory
            .selectFrom(member)
            .orderBy(member.username.desc())
            .offset(1)
            .limit(2)
            .fetch();

    assertThat(result.size()).isEqualTo(2);
}
```

- 집합

원하는 값을 조회했다면 Tuple로 반환된다. Querydsl이 제공하는 기능이다.

```java
@Test
public void aggregation() throws Exception {
    List<Tuple> result = queryFactory
            .select(member.count(),
                    member.age.sum(),
                    member.age.avg(),
                    member.age.max(),
                    member.age.min())
            .from(member)
            .fetch();

    Tuple tuple = result.get(0);
    assertThat(tuple.get(member.count())).isEqualTo(4);
    assertThat(tuple.get(member.age.sum())).isEqualTo(100);
    assertThat(tuple.get(member.age.avg())).isEqualTo(25);
    assertThat(tuple.get(member.age.max())).isEqualTo(40);
    assertThat(tuple.get(member.age.min())).isEqualTo(10);
}
```

DTO를 사용한 방법을 Tuple보다 더 많이 사용한다.

- groupby

```java
@Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();
        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);
        assertThat(teamA.get(team.name)).isEqualTo("teamA");
        assertThat(teamA.get(member.age.avg())).isEqualTo(15);
        assertThat(teamB.get(team.name)).isEqualTo("teamB");
        assertThat(teamB.get(member.age.avg())).isEqualTo(35);
    }
```

having역시 적용 가능하다.

- join

```java
@Test
public void join() throws Exception {
    QMember member = QMember.member;
    List<Member> result = queryFactory
            .selectFrom(member)
            .join(member.team, team)
            .where(team.name.eq("teamA"))
            .fetch();
    assertThat(result)
            .extracting("username")
            .containsExactly("member1", "member2");
}
```

**연관관계가 없는 경우도 theta join이 가능하다.**

```java
@Test
public void theta_join() throws Exception {//연관관계 없이도 join가능
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    List<Member> result = queryFactory
            .select(member)
            .from(member, team) //theta 조인 방법
            .where(member.username.eq(team.name))
            .fetch();
    assertThat(result)
            .extracting("username")
            .containsExactly("teamA", "teamB");

}
```

from에서 join할 대상 테이블을 적고 where절에서 그 기준(on)을 처리하면 된다고 보면된다.

다만, theta join은 outer join이 불가능하다. → 그러나, `on`을 사용하면 가능하다.

- on 사용

```java
@Test
public void join_on_filtering() throws Exception {
    List<Tuple> result = queryFactory
            .select(member, team)
            .from(member)
            .leftJoin(member.team, team)
						.on(team.name.eq("teamA"))
            .orderBy(team.name.desc().nullsLast())
            .fetch();

    for(int i=0; i< result.size(); i++){
        if(i<2){
            Assertions.assertThat(result.get(i).get(team).getName()).isEqualTo("teamA");
        }else{
            final int index = i;
            org.junit.jupiter.api.Assertions.assertThrows(NullPointerException.class, ()->
                    result.get(index).get(team).getName());

        }
    }

 }
```

teamA인 것만 가져오는데 left join이니까 teamB일때는 Member에 대한 값만 가져오게된다.

참고 : local variables referenced from a lambda expression must be final or effectively final

[자바의 effectively final](https://madplay.github.io/post/effectively-final-in-java)

**연관관계가 없는 theta join**

```java
@Test
public void join_on_no_relation() throws Exception {
    em.persist(new Member("teamA"));
    em.persist(new Member("teamB"));
    List<Tuple> result = queryFactory
            **.select(member, team)**
            .from(member)
            **.leftJoin(team)**.on(member.username.eq(team.name))
            .orderBy(team.name.asc().nullsLast())
            .fetch();

     for(int i=0; i< result.size(); i++){
         if(i<=1) {
             Assertions.assertThat(result.get(i).get(team).getName()).isNotNull();
         }
         else{//member의 이릌과 team의 이름이 다른 4가지 미리 넣어둔 경우
             final int index = i;
             org.junit.jupiter.api.Assertions.assertThrows(NullPointerException.class, ()->
                     result.get(index).get(team).getName());

         }
     }
}
```

- 패치조인

```java
@Test
public void fetchJoinUse() throws Exception {
    em.flush();
    em.clear();//영속성 컨텍스를 날리고 진행

    Member findMember = queryFactory
            .selectFrom(member)
            .join(member.team, team).fetchJoin()
            .where(member.username.eq("member1"))
            .fetchOne();

    boolean loaded =
            emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());
    assertThat(loaded).as("페치 조인 적용").isTrue();
}
```

- 서브쿼리

```java
@Test
  public void subQuery() throws Exception {
      QMember memberSub = new QMember("memberSub");
      List<Member> result = queryFactory
              .selectFrom(member)
              .where(member.age.eq(
                      JPAExpressions
                              .select(memberSub.age.max())
                              .from(memberSub)
              )) .fetch();
      assertThat(result).extracting("age")
              .containsExactly(40);
  }
```

**in을 사용한 subquery의 예제도 보자!**

```java
@Test
    public void subQueryIn() throws Exception {
        QMember memberSub = new QMember("memberSub");
        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.in(
                        JPAExpressions
                                .select(memberSub.age)
                                .from(memberSub)
                                .where(memberSub.age.gt(10))
                )) .fetch();
        assertThat(result).extracting("age")
                .containsExactly(20, 30, 40);
    }
```


**다만, from절의 서브쿼리(인라인뷰)는 지원하지 않는다.**

**해결방안**

1. 서브쿼리를 join으로 변경한다.
2. 애플리케이션에서 쿼리를 2번 분리해서 실행한다. → 성능고려 필수
3. nativeSQL을 사용한다.

⇒ 1~3번중 하나를 사용한다.

- case절

다만, DB에서 case문을 정말 사용해야하는가?를 고민해야함

로직은 DB쪽에서 처리하는게 좋지 않을 수 있다.

따라서, 사실 아래의 예제는 application logic에서 처리 권고한다.

```java
@Test void complexCase(){
    List<String> result = queryFactory
            .select(new CaseBuilder()
                    .when(member.age.between(0, 20)).then("0~20살")
                    .when(member.age.between(21, 30)).then("21~30살")
                    .otherwise("기타"))
            .from(member)
            .fetch();
}
```
